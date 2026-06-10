<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  title: {
    type: String,
    required: true
  },
  artist: {
    type: String,
    required: true
  },
  artStyle: {
    type: String, // 'flow', 'fractal', 'spirograph', 'chaos', 'mesh', 'cubes'
    default: 'flow'
  },
  palette: {
    type: String, // 'coral', 'blue', 'yellow', 'pink'
    default: 'coral'
  }
})

const cardCanvasRef = ref<HTMLCanvasElement | null>(null)
let animationFrameId: number
let time = 0

// Map palettes to actual colors
const paletteColors = {
  coral: {
    primary: '#FF6B6B',
    secondary: 'rgba(255, 107, 107, 0.6)',
    glow: 'rgba(255, 107, 107, 0.45)',
    accent: 'rgba(255, 139, 148, 0.3)'
  },
  blue: {
    primary: '#4ECDC4',
    secondary: 'rgba(78, 205, 196, 0.6)',
    glow: 'rgba(78, 205, 196, 0.45)',
    accent: 'rgba(255, 230, 109, 0.3)'
  },
  yellow: {
    primary: '#FFE66D',
    secondary: 'rgba(255, 230, 109, 0.6)',
    glow: 'rgba(255, 230, 109, 0.45)',
    accent: 'rgba(78, 205, 196, 0.3)'
  },
  pink: {
    primary: '#FF8B94',
    secondary: 'rgba(255, 139, 148, 0.6)',
    glow: 'rgba(255, 139, 148, 0.45)',
    accent: 'rgba(255, 107, 107, 0.3)'
  }
}

const colors = paletteColors[props.palette as keyof typeof paletteColors] || paletteColors.coral

// Render loop for different generative art styles
const runGenerator = () => {
  const canvas = cardCanvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  // Set crisp canvas dimensions
  const dpr = window.devicePixelRatio || 1
  const width = 360
  const height = 240
  canvas.width = width * dpr
  canvas.height = height * dpr
  canvas.style.width = '100%'
  canvas.style.height = '100%'
  ctx.scale(dpr, dpr)

  // Pre-generate static parameters to keep artwork cohesive
  const seed = Math.random() * 100
  const speed = 0.005 + Math.random() * 0.005

  // Initialize particles for flow fields and attractors
  interface Point {
    x: number
    y: number
    vx: number
    vy: number
    life: number
    maxLife: number
    color: string
  }

  const particles: Point[] = []
  if (props.artStyle === 'flow' || props.artStyle === 'chaos') {
    for (let i = 0; i < 40; i++) {
      particles.push({
        x: Math.random() * width,
        y: Math.random() * height,
        vx: 0,
        vy: 0,
        life: Math.random() * 100,
        maxLife: 100 + Math.random() * 100,
        color: Math.random() > 0.4 ? colors.primary : colors.secondary
      })
    }
  }

  // Constellation Mesh points
  const meshPoints: Point[] = []
  if (props.artStyle === 'mesh') {
    for (let i = 0; i < 18; i++) {
      meshPoints.push({
        x: Math.random() * width,
        y: Math.random() * height,
        vx: (Math.random() - 0.5) * 0.8,
        vy: (Math.random() - 0.5) * 0.8,
        life: 0,
        maxLife: 0,
        color: ''
      })
    }
  }

  const render = () => {
    time += speed

    // Choose render routine based on artStyle
    if (props.artStyle === 'flow') {
      // Flow field style: Accumulative trails
      // Instead of clearing completely, overlay transparent background for trails
      ctx.fillStyle = 'rgba(255, 255, 255, 0.03)'
      ctx.fillRect(0, 0, width, height)

      particles.forEach((p) => {
        p.life++
        if (p.life > p.maxLife) {
          p.x = Math.random() * width
          p.y = Math.random() * height
          p.life = 0
        }

        // Mathematical vector fields using sin/cos of coordinate + time
        const angle = Math.sin(p.x * 0.006 + seed) * Math.PI * 2 + Math.cos(p.y * 0.006 + time) * Math.PI
        p.vx = Math.cos(angle) * 1.5
        p.vy = Math.sin(angle) * 1.5

        ctx.beginPath()
        ctx.strokeStyle = p.color
        ctx.lineWidth = 1.2
        ctx.lineCap = 'round'
        ctx.moveTo(p.x, p.y)
        ctx.lineTo(p.x + p.vx, p.y + p.vy)
        ctx.stroke()

        p.x += p.vx
        p.y += p.vy
      })

    } else if (props.artStyle === 'fractal') {
      // Rotating Golden Ratio Fractal Mandala
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, width, height)

      const cx = width / 2
      const cy = height / 2
      const maxRadius = 100
      const branches = 6 + Math.floor(seed % 4)

      ctx.save()
      ctx.translate(cx, cy)
      ctx.rotate(time * 0.1)

      for (let i = 0; i < branches; i++) {
        ctx.rotate((Math.PI * 2) / branches)
        ctx.beginPath()
        ctx.strokeStyle = colors.primary
        ctx.lineWidth = 0.8

        let rx = 0
        let ry = 0

        for (let j = 0; j < 60; j++) {
          const t = j * 0.15
          const r = t * 1.6 * (1 + Math.sin(time + t) * 0.15)
          rx = Math.cos(t) * r
          ry = Math.sin(t) * r
          if (j === 0) ctx.moveTo(rx, ry)
          else ctx.lineTo(rx, ry)
        }
        ctx.stroke()

        // Tiny floating accent circles on mandala nodes
        ctx.fillStyle = colors.secondary
        ctx.beginPath()
        ctx.arc(rx, ry, 3, 0, Math.PI * 2)
        ctx.fill()
      }
      ctx.restore()

    } else if (props.artStyle === 'spirograph') {
      // Spirograph / Epicycloid wave generator
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, width, height)

      const cx = width / 2
      const cy = height / 2
      ctx.lineWidth = 1.0

      // Outer radius, inner radius, pen distance
      const R = 75
      const r = 45 + Math.sin(time * 0.5) * 10
      const p = 35 + Math.cos(time * 0.2) * 15

      ctx.beginPath()
      ctx.strokeStyle = colors.primary
      for (let t = 0; t < Math.PI * 10; t += 0.05) {
        const x = cx + (R - r) * Math.cos(t) + p * Math.cos(((R - r) * t) / r)
        const y = cy + (R - r) * Math.sin(t) - p * Math.sin(((R - r) * t) / r)
        if (t === 0) ctx.moveTo(x, y)
        else ctx.lineTo(x, y)
      }
      ctx.stroke()

      // Inner layered spinner
      ctx.beginPath()
      ctx.strokeStyle = colors.accent
      for (let t = 0; t < Math.PI * 6; t += 0.05) {
        const x = cx + (R/2 - r/2) * Math.cos(t + time) + (p/2) * Math.cos(((R/2 - r/2) * t) / (r/2))
        const y = cy + (R/2 - r/2) * Math.sin(t + time) - (p/2) * Math.sin(((R/2 - r/2) * t) / (r/2))
        if (t === 0) ctx.moveTo(x, y)
        else ctx.lineTo(x, y)
      }
      ctx.stroke()

    } else if (props.artStyle === 'chaos') {
      // Chaos game / Clifford Attractor-like vortex particles
      ctx.fillStyle = 'rgba(255, 255, 255, 0.04)'
      ctx.fillRect(0, 0, width, height)

      particles.forEach((p) => {
        const dx = width / 2 - p.x
        const dy = height / 2 - p.y
        const dist = Math.hypot(dx, dy)

        // Swirling force towards center
        const angle = Math.atan2(dy, dx) + Math.PI / 2 + 0.15
        p.vx = Math.cos(angle) * 1.5 + (dx / dist) * 0.2
        p.vy = Math.sin(angle) * 1.5 + (dy / dist) * 0.2

        ctx.fillStyle = p.color
        ctx.beginPath()
        ctx.arc(p.x, p.y, 1.5 + Math.sin(time + dist * 0.05) * 1, 0, Math.PI * 2)
        ctx.fill()

        p.x += p.vx
        p.y += p.vy

        // If too close to center, respawn at edge
        if (dist < 10) {
          const spawnAngle = Math.random() * Math.PI * 2
          p.x = width / 2 + Math.cos(spawnAngle) * 110
          p.y = height / 2 + Math.sin(spawnAngle) * 110
        }
      })

    } else if (props.artStyle === 'mesh') {
      // Molecular / Constellation dynamic mesh
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, width, height)

      // Move points
      meshPoints.forEach((p) => {
        p.x += p.vx
        p.y += p.vy

        if (p.x < 0 || p.x > width) p.vx *= -1
        if (p.y < 0 || p.y > height) p.vy *= -1

        ctx.fillStyle = colors.primary
        ctx.beginPath()
        ctx.arc(p.x, p.y, 3, 0, Math.PI * 2)
        ctx.fill()
      })

      // Draw connections
      ctx.lineWidth = 0.5
      for (let i = 0; i < meshPoints.length; i++) {
        for (let j = i + 1; j < meshPoints.length; j++) {
          const p1 = meshPoints[i]
          const p2 = meshPoints[j]
          const dist = Math.hypot(p2.x - p1.x, p2.y - p1.y)

          if (dist < 65) {
            ctx.strokeStyle = `rgba(${parseInt(colors.primary.slice(1,3), 16)}, ${parseInt(colors.primary.slice(3,5), 16)}, ${parseInt(colors.primary.slice(5,7), 16)}, ${1 - dist / 65})`
            ctx.beginPath()
            ctx.moveTo(p1.x, p1.y)
            ctx.lineTo(p2.x, p2.y)
            ctx.stroke()
          }
        }
      }

    } else if (props.artStyle === 'cubes') {
      // Isometric pulsing cubes / geometric matrix
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, width, height)

      const cols = 8
      const rows = 5
      const sizeX = width / cols
      const sizeY = height / rows

      for (let c = 0; c < cols; c++) {
        for (let r = 0; r < rows; r++) {
          const px = c * sizeX + sizeX / 2
          const py = r * sizeY + sizeY / 2
          const wave = Math.sin(time + c * 0.5 + r * 0.5)

          ctx.save()
          ctx.translate(px, py)
          ctx.rotate(time * 0.2 + (c + r) * 0.1)

          // Size oscillation
          const scale = 0.5 + wave * 0.35
          ctx.scale(scale, scale)

          ctx.strokeStyle = colors.primary
          ctx.lineWidth = 1.0
          ctx.strokeRect(-sizeX/3, -sizeY/3, (sizeX * 2)/3, (sizeY * 2)/3)

          ctx.fillStyle = colors.accent
          ctx.fillRect(-sizeX/6, -sizeY/6, sizeX/3, sizeY/3)

          ctx.restore()
        }
      }
    }

    animationFrameId = requestAnimationFrame(render)
  }

  render()

  onUnmounted(() => {
    cancelAnimationFrame(animationFrameId)
  })
}

onMounted(() => {
  runGenerator()
})
</script>

<template>
  <div
    class="relative flex flex-col overflow-hidden rounded-2xl border border-white/20 bg-white/10 p-4 shadow-sm backdrop-blur-[10px] transition-all duration-500 ease-out hover:-translate-y-2 hover:scale-[1.03] group"
    :style="{
      '--hover-glow-color': colors.glow
    }"
  >
    <!-- Live Canvas Container -->
    <div class="relative w-full aspect-[3/2] overflow-hidden rounded-xl border border-white/10 bg-slate-50/50">
      <canvas
        ref="cardCanvasRef"
        class="w-full h-full object-cover rounded-xl"
      />
      <!-- Shimmer and Grid overlay on Card image -->
      <div class="absolute inset-0 bg-gradient-to-t from-white/10 to-transparent mix-blend-overlay pointer-events-none" />
      <div class="absolute inset-0 bg-[radial-gradient(ellipse_at_top_right,var(--hover-glow-color),transparent_65%)] opacity-30 group-hover:opacity-60 transition-opacity duration-500 pointer-events-none" />
    </div>

    <!-- Art Details -->
    <div class="mt-4 flex flex-col">
      <h3 class="text-lg font-bold text-gray-900 group-hover:text-black transition-colors duration-300">
        {{ title }}
      </h3>
      <p class="text-sm font-medium text-gray-600/95 mt-1">
        By {{ artist }}
      </p>
      
      <!-- Mini Pill with Art Style -->
      <div class="mt-3 flex items-center justify-between">
        <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-semibold uppercase tracking-wider bg-white/40 border border-white/30 text-gray-800">
          {{ artStyle }}
        </span>
        <button
          class="flex items-center justify-center w-8 h-8 rounded-full bg-white/50 hover:bg-white border border-white/40 text-gray-800 hover:text-black shadow-sm transition-all duration-300 active:scale-95"
          aria-label="View artwork details"
        >
          <UIcon name="i-lucide-arrow-right" class="w-4 h-4" />
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* High-performance hardware-accelerated drop shadow glow trigger on hover */
.group:hover {
  border-color: rgba(255, 255, 255, 0.4);
  box-shadow: 0 20px 40px -15px var(--hover-glow-color), 
              0 0 50px -10px var(--hover-glow-color),
              inset 0 1px 0 rgba(255, 255, 255, 0.4);
}
</style>
