<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)

interface Blob {
  x: number
  y: number
  vx: number
  vy: number
  radius: number
  color: string
  targetRadius: number
  angle: number
  speed: number
}

interface Particle {
  x: number
  y: number
  vx: number
  vy: number
  size: number
  color: string
  alpha: number
  decay: number
  angle: number
  amplitude: number
}

const colors = [
  'rgba(255, 107, 107, 0.45)',  // Coral #FF6B6B
  'rgba(78, 205, 196, 0.45)',   // Electric Blue #4ECDC4
  'rgba(255, 230, 109, 0.45)',  // Golden Yellow #FFE66D
  'rgba(255, 139, 148, 0.45)'   // Hot Pink #FF8B94
]

const particleColors = [
  'rgba(255, 107, 107, 0.25)',
  'rgba(78, 205, 196, 0.25)',
  'rgba(255, 230, 109, 0.25)',
  'rgba(255, 139, 148, 0.25)'
]

let animationFrameId: number
let blobs: Blob[] = []
let particles: Particle[] = []

const mouse = {
  x: 0,
  y: 0,
  targetX: 0,
  targetY: 0,
  currentX: 0,
  currentY: 0,
  isActive: false
}

const initCanvas = () => {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const resizeCanvas = () => {
    canvas.width = window.innerWidth
    canvas.height = window.innerHeight
    setupBlobsAndParticles()
  }

  const setupBlobsAndParticles = () => {
    const w = canvas.width
    const h = canvas.height
    const minDim = Math.min(w, h)

    // Setup 4 large gradient blobs (one for each color)
    blobs = colors.map((color, index) => {
      const radius = minDim * (0.25 + Math.random() * 0.15) // 25% to 40% of viewport min dimension
      return {
        x: Math.random() * w,
        y: Math.random() * h,
        vx: (Math.random() - 0.5) * 1.5,
        vy: (Math.random() - 0.5) * 1.5,
        radius,
        targetRadius: radius,
        color,
        angle: Math.random() * Math.PI * 2,
        speed: 0.005 + Math.random() * 0.01
      }
    })

    // Setup particles
    particles = []
    const particleCount = Math.min(60, Math.floor((w * h) / 30000)) // Scaled by screen size
    for (let i = 0; i < particleCount; i++) {
      particles.push({
        x: Math.random() * w,
        y: Math.random() * h,
        vx: (Math.random() - 0.5) * 0.5,
        vy: -0.2 - Math.random() * 0.6, // Float upwards
        size: 3 + Math.random() * 8,
        color: particleColors[Math.floor(Math.random() * particleColors.length)],
        alpha: 0.1 + Math.random() * 0.4,
        decay: 0.0005 + Math.random() * 0.001,
        angle: Math.random() * Math.PI * 2,
        amplitude: 0.2 + Math.random() * 0.8
      })
    }
  }

  resizeCanvas()
  window.addEventListener('resize', resizeCanvas)

  const handleMouseMove = (e: MouseEvent) => {
    mouse.targetX = e.clientX
    mouse.targetY = e.clientY
    mouse.isActive = true
  }

  const handleMouseLeave = () => {
    mouse.isActive = false
  }

  window.addEventListener('mousemove', handleMouseMove)
  document.addEventListener('mouseleave', handleMouseLeave)

  // Initialize mouse in center
  mouse.targetX = window.innerWidth / 2
  mouse.targetY = window.innerHeight / 2
  mouse.currentX = mouse.targetX
  mouse.currentY = mouse.targetY

  const animate = () => {
    const w = canvas.width
    const h = canvas.height

    // Smoothly interpolate mouse position (lerp)
    mouse.currentX += (mouse.targetX - mouse.currentX) * 0.03
    mouse.currentY += (mouse.targetY - mouse.currentY) * 0.03

    // Clear with a very slight translucent off-white for a beautiful trail/dreamy overlay
    // but drawing a complete solid off-white under it to avoid browser trail compounding.
    ctx.fillStyle = '#fbfbfc' // Bright, elegant off-white
    ctx.fillRect(0, 0, w, h)

    // Draw grid overlay for extra artistic structure
    ctx.strokeStyle = 'rgba(0, 0, 0, 0.015)'
    ctx.lineWidth = 1
    const gridSize = 80
    for (let x = 0; x < w; x += gridSize) {
      ctx.beginPath()
      ctx.moveTo(x, 0)
      ctx.lineTo(x, h)
      ctx.stroke()
    }
    for (let y = 0; y < h; y += gridSize) {
      ctx.beginPath()
      ctx.moveTo(0, y)
      ctx.lineTo(w, y)
      ctx.stroke()
    }

    // Move and Draw Blobs
    blobs.forEach((blob) => {
      // Periodic size oscillation
      blob.angle += blob.speed
      blob.radius = blob.targetRadius + Math.sin(blob.angle) * 30

      // Slowly float
      blob.x += blob.vx
      blob.y += blob.vy

      // Reactive mouse shifting: gently push blobs away from mouse or shift their center
      if (mouse.isActive) {
        const dx = mouse.currentX - blob.x
        const dy = mouse.currentY - blob.y
        const dist = Math.hypot(dx, dy)
        const maxDist = Math.max(w, h) * 0.4

        if (dist < maxDist) {
          const force = (1 - dist / maxDist) * 0.3
          // Push gently away from mouse
          blob.x -= (dx / dist) * force * 12
          blob.y -= (dy / dist) * force * 12
        }
      }

      // Keep within boundaries (soft bounds with cushion)
      const cushion = blob.radius * 0.5
      if (blob.x < -cushion) {
        blob.x = -cushion
        blob.vx *= -1
      } else if (blob.x > w + cushion) {
        blob.x = w + cushion
        blob.vx *= -1
      }
      if (blob.y < -cushion) {
        blob.y = -cushion
        blob.vy *= -1
      } else if (blob.y > h + cushion) {
        blob.y = h + cushion
        blob.vy *= -1
      }

      // Create a shifting focal point of the gradient based on mouse position
      // This solves: "Mouse position should slowly shift the gradient direction on the background canvas"
      const mouseOffsetX = ((mouse.currentX - w / 2) / (w / 2)) * 30
      const mouseOffsetY = ((mouse.currentY - h / 2) / (h / 2)) * 30

      const gradX = blob.x + mouseOffsetX
      const gradY = blob.y + mouseOffsetY

      const grad = ctx.createRadialGradient(
        gradX, gradY, 0,
        blob.x, blob.y, blob.radius
      )
      grad.addColorStop(0, blob.color)
      grad.addColorStop(0.5, blob.color.replace('0.45', '0.15'))
      grad.addColorStop(1, 'rgba(255, 255, 255, 0)')

      ctx.fillStyle = grad
      ctx.beginPath()
      ctx.arc(blob.x, blob.y, blob.radius, 0, Math.PI * 2)
      ctx.fill()
    })

    // Move and Draw Particles
    particles.forEach((p) => {
      p.angle += 0.01
      // Floating upwards with wave oscillation
      p.x += p.vx + Math.sin(p.angle) * p.amplitude * 0.3
      p.y += p.vy

      // Mouse influence on particles: repelled gently
      if (mouse.isActive) {
        const dx = mouse.currentX - p.x
        const dy = mouse.currentY - p.y
        const dist = Math.hypot(dx, dy)
        const activeDist = 120

        if (dist < activeDist) {
          const force = (1 - dist / activeDist) * 0.8
          p.x -= (dx / dist) * force * 3
          p.y -= (dy / dist) * force * 3
        }
      }

      // Recycle particles if they go off top or sides
      if (p.y < -p.size || p.x < -p.size || p.x > w + p.size) {
        p.y = h + p.size + Math.random() * 20
        p.x = Math.random() * w
        p.angle = Math.random() * Math.PI * 2
      }

      // Particle rendering with glowing soft radial falloff
      const grad = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, p.size)
      grad.addColorStop(0, p.color)
      grad.addColorStop(1, 'rgba(255, 255, 255, 0)')

      ctx.fillStyle = grad
      ctx.beginPath()
      ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2)
      ctx.fill()
    })

    animationFrameId = requestAnimationFrame(animate)
  }

  animate()

  onUnmounted(() => {
    window.removeEventListener('resize', resizeCanvas)
    window.removeEventListener('mousemove', handleMouseMove)
    document.removeEventListener('mouseleave', handleMouseLeave)
    cancelAnimationFrame(animationFrameId)
  })
}

onMounted(() => {
  initCanvas()
})
</script>

<template>
  <canvas
    ref="canvasRef"
    class="fixed inset-0 w-full h-full pointer-events-none -z-10"
  />
</template>
