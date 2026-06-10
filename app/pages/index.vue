<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Curated generative artwork catalog
const artworks = [
  { id: 1, title: 'Aura Fields', artist: 'Evelyn Sterling', style: 'flow', palette: 'coral' },
  { id: 2, title: 'Golden Lattice', artist: 'Marcus Vance', style: 'fractal', palette: 'yellow' },
  { id: 3, title: 'Cypher Web', artist: 'Aria Thorne', style: 'spirograph', palette: 'blue' },
  { id: 4, title: 'Cosmic Attractor', artist: 'Julian Cross', style: 'chaos', palette: 'pink' },
  { id: 5, title: 'Nexus Organica', artist: 'Elena Rostova', style: 'mesh', palette: 'blue' },
  { id: 6, title: 'Isometric Dreams', artist: 'Sora Takahashi', style: 'cubes', palette: 'coral' },
  { id: 7, title: 'Ethereal Waves', artist: 'Nils Lindstrom', style: 'flow', palette: 'pink' },
  { id: 8, title: 'Mandala of Light', artist: 'Zara Patel', style: 'fractal', palette: 'yellow' },
  { id: 9, title: 'Hyperbolic Spiro', artist: 'Dr. Theo Kern', style: 'spirograph', palette: 'blue' }
]

// Category Filter ref
const activeFilter = ref('all')
const filteredArtworks = computed(() => {
  if (activeFilter.value === 'all') return artworks
  return artworks.filter(art => art.style === activeFilter.value)
})

// Featured Interactive Canvas Logic
const featuredCanvasRef = ref<HTMLCanvasElement | null>(null)
let featuredAnimId: number
let fTime = 0

interface OrbitParticle {
  x: number
  y: number
  originX: number
  originY: number
  vx: number
  vy: number
  radius: number
  color: string
  angle: number
  speed: number
  orbitRadius: number
}

const fMouse = {
  x: 0,
  y: 0,
  targetX: 0,
  targetY: 0,
  active: false
}

const initFeaturedArt = () => {
  const canvas = featuredCanvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  if (!ctx) return

  const resize = () => {
    canvas.width = canvas.parentElement?.clientWidth || window.innerWidth
    canvas.height = canvas.parentElement?.clientHeight || 600
    setupParticles()
  }

  const fColors = ['#FF6B6B', '#4ECDC4', '#FFE66D', '#FF8B94']
  let fParticles: OrbitParticle[] = []

  const setupParticles = () => {
    fParticles = []
    const count = 350
    const cx = canvas.width / 2
    const cy = canvas.height / 2

    for (let i = 0; i < count; i++) {
      const angle = Math.random() * Math.PI * 2
      const orbitRadius = 40 + Math.random() * Math.min(canvas.width, canvas.height) * 0.4
      fParticles.push({
        x: cx + Math.cos(angle) * orbitRadius,
        y: cy + Math.sin(angle) * orbitRadius,
        originX: cx,
        originY: cy,
        vx: 0,
        vy: 0,
        radius: 1 + Math.random() * 2.5,
        color: fColors[Math.floor(Math.random() * fColors.length)],
        angle,
        speed: 0.005 + Math.random() * 0.01,
        orbitRadius
      })
    }
  }

  resize()
  window.addEventListener('resize', resize)

  const handleMouseMove = (e: MouseEvent) => {
    const rect = canvas.getBoundingClientRect()
    fMouse.targetX = e.clientX - rect.left
    fMouse.targetY = e.clientY - rect.top
    fMouse.active = true
  }

  const handleMouseLeave = () => {
    fMouse.active = false
  }

  canvas.addEventListener('mousemove', handleMouseMove)
  canvas.addEventListener('mouseleave', handleMouseLeave)

  // Prime mouse coords
  fMouse.targetX = canvas.width / 2
  fMouse.targetY = canvas.height / 2
  fMouse.x = fMouse.targetX
  fMouse.y = fMouse.targetY

  const draw = () => {
    fTime += 0.01
    ctx.fillStyle = 'rgba(10, 15, 29, 0.08)' // Dark slate background to let the glowing orbits stand out
    ctx.fillRect(0, 0, canvas.width, canvas.height)

    // Lerp mouse
    fMouse.x += (fMouse.targetX - fMouse.x) * 0.05
    fMouse.y += (fMouse.targetY - fMouse.y) * 0.05

    // Draw central gravity core
    const coreGrad = ctx.createRadialGradient(fMouse.x, fMouse.y, 0, fMouse.x, fMouse.y, 60)
    coreGrad.addColorStop(0, 'rgba(78, 205, 196, 0.25)')
    coreGrad.addColorStop(1, 'rgba(10, 15, 29, 0)')
    ctx.fillStyle = coreGrad
    ctx.beginPath()
    ctx.arc(fMouse.x, fMouse.y, 60, 0, Math.PI * 2)
    ctx.fill()

    fParticles.forEach((p) => {
      // Swarm algorithm around the mouse pointer
      p.angle += p.speed
      
      const targetX = fMouse.x + Math.cos(p.angle) * p.orbitRadius
      const targetY = fMouse.y + Math.sin(p.angle) * p.orbitRadius

      p.vx += (targetX - p.x) * 0.02
      p.vy += (targetY - p.y) * 0.02

      // Friction
      p.vx *= 0.92
      p.vy *= 0.92

      p.x += p.vx
      p.y += p.vy

      // Orbit trail or connection line to nearest neighborhood if within range
      ctx.fillStyle = p.color
      ctx.beginPath()
      // Size dynamic depending on speed
      const speedMag = Math.hypot(p.vx, p.vy)
      ctx.arc(p.x, p.y, p.radius + speedMag * 0.1, 0, Math.PI * 2)
      ctx.fill()
    })

    featuredAnimId = requestAnimationFrame(draw)
  }

  draw()

  onUnmounted(() => {
    window.removeEventListener('resize', resize)
    canvas.removeEventListener('mousemove', handleMouseMove)
    canvas.removeEventListener('mouseleave', handleMouseLeave)
    cancelAnimationFrame(featuredAnimId)
  })
}

onMounted(() => {
  initFeaturedArt()
})
</script>

<template>
  <div class="relative w-full">
    
    <!-- Hero Section -->
    <section
      id="hero"
      class="relative min-h-screen flex flex-col items-center justify-center px-6 overflow-hidden"
    >
      <div class="relative z-10 max-w-5xl mx-auto flex flex-col items-center justify-center text-center">
        <!-- Floating Glass Badge -->
        <div class="inline-flex items-center space-x-2 px-4 py-1.5 rounded-full border border-white/40 bg-white/20 backdrop-blur-md shadow-sm mb-6 animate-pulse">
          <span class="w-2.5 h-2.5 rounded-full bg-coral"></span>
          <span class="text-xs font-bold tracking-widest text-gray-800 uppercase">Interactive Exhibition</span>
        </div>

        <!-- Title CANVAS -->
        <h1 class="text-5xl min-[360px]:text-6xl min-[410px]:text-7xl sm:text-[9rem] md:text-[12rem] lg:text-[14rem] font-black tracking-tighter leading-none select-none text-white drop-shadow-[0_20px_50px_rgba(10,15,29,0.18)] mb-6 uppercase transition-transform duration-700 hover:scale-[1.01]">
          CANVAS
        </h1>

        <!-- Tagline -->
        <p class="max-w-2xl text-base sm:text-lg md:text-2xl font-medium text-gray-800/90 leading-relaxed tracking-wide drop-shadow-sm px-4">
          A living, breathing digital gallery that manifests algorithmic beauty, synthesized in real time on the fabric of your browser.
        </p>

        <!-- Quick actions -->
        <div class="mt-8 sm:mt-10 flex flex-wrap items-center justify-center gap-3.5 sm:gap-4">
          <UButton
            to="#gallery"
            size="lg"
            class="rounded-full bg-coral hover:bg-coral/90 text-white font-bold shadow-lg shadow-coral/35 px-6 py-3 sm:px-8 sm:py-3.5 transition-all duration-300 hover:scale-105 active:scale-95 text-sm sm:text-base"
          >
            Enter Gallery
          </UButton>
          <UButton
            to="#featured"
            size="lg"
            color="neutral"
            variant="subtle"
            class="rounded-full bg-white/55 hover:bg-white border border-white/60 text-gray-900 font-bold px-6 py-3 sm:px-8 sm:py-3.5 backdrop-blur-md transition-all duration-300 hover:scale-105 active:scale-95 text-sm sm:text-base"
          >
            Interact Now
          </UButton>
        </div>
      </div>

      <!-- Scroll Indicator -->
      <a
        href="#gallery"
        class="absolute bottom-10 left-1/2 -translate-x-1/2 z-10 flex flex-col items-center space-y-2 text-gray-800/85 hover:text-coral transition-colors duration-300 group"
        aria-label="Scroll down to gallery"
      >
        <span class="text-xs font-bold tracking-widest uppercase">Scroll to Discover</span>
        <div class="w-6 h-10 rounded-full border-2 border-gray-800/80 flex justify-center p-1.5 group-hover:border-coral transition-colors duration-300">
          <div class="w-1.5 h-1.5 rounded-full bg-gray-800 group-hover:bg-coral animate-bounce"></div>
        </div>
      </a>
    </section>

    <!-- Curated Gallery Section -->
    <section
      id="gallery"
      class="relative py-16 sm:py-28 px-6 md:px-12 max-w-7xl mx-auto"
    >
      <div class="flex flex-col md:flex-row md:items-end justify-between mb-12 sm:mb-16 border-b border-gray-900/10 pb-8">
        <div>
          <h2 class="text-3xl sm:text-4xl md:text-5xl font-black text-gray-900 tracking-tight uppercase">
            Curated Works
          </h2>
          <p class="text-base sm:text-lg text-gray-600/95 mt-3 max-w-xl">
            Each artifact below is computed natively and compiled in real time. Click on any piece to appreciate its procedural geometry.
          </p>
        </div>

        <!-- Filter Pills -->
        <div class="mt-8 md:mt-0 flex overflow-x-auto md:overflow-visible flex-nowrap md:flex-wrap gap-2.5 max-w-full pb-3 md:pb-0 -mx-6 px-6 md:mx-0 md:px-0 scrollbar-none">
          <button
            v-for="filter in ['all', 'flow', 'fractal', 'spirograph', 'chaos', 'mesh', 'cubes']"
            :key="filter"
            @click="activeFilter = filter"
            class="px-4 py-2 rounded-full text-xs font-extrabold uppercase tracking-widest transition-all duration-300 flex-shrink-0"
            :class="[
              activeFilter === filter
                ? 'bg-gray-950 text-white shadow-md scale-105'
                : 'bg-white/40 border border-white/40 text-gray-700 hover:bg-white/70 hover:text-black'
            ]"
          >
            {{ filter }}
          </button>
        </div>
      </div>

      <!-- Masonry Grid with Columns -->
      <div class="columns-1 sm:columns-2 lg:columns-3 gap-6 space-y-6">
        <ArtCard
          v-for="art in filteredArtworks"
          :key="art.id"
          :title="art.title"
          :artist="art.artist"
          :artStyle="art.style"
          :palette="art.palette"
          class="break-inside-avoid"
        />
      </div>
    </section>

    <!-- Fullscreen Featured Section -->
    <section
      id="featured"
      class="relative w-full h-[700px] md:h-[800px] flex items-center justify-start overflow-hidden bg-brand-dark"
    >
      <!-- Real-time particle storm simulation canvas spanning full feature area -->
      <canvas
        ref="featuredCanvasRef"
        class="absolute inset-0 w-full h-full object-cover"
      />

      <div class="absolute inset-0 bg-gradient-to-r from-brand-dark via-brand-dark/40 to-transparent pointer-events-none" />

      <!-- Overlaid Glassmorphic Description Card -->
      <div class="relative z-10 max-w-2xl ml-6 md:ml-20 mr-6 p-8 md:p-12 rounded-3xl border border-white/10 bg-brand-dark/65 shadow-2xl backdrop-blur-xl">
        <div class="inline-flex items-center space-x-2 px-3 py-1 rounded-full border border-electric-blue/40 bg-electric-blue/10 mb-6">
          <span class="w-2 h-2 rounded-full bg-electric-blue animate-ping"></span>
          <span class="text-[10px] font-black tracking-widest text-electric-blue uppercase">Featured Piece</span>
        </div>

        <h2 class="text-3xl md:text-5xl font-black text-white tracking-tight uppercase">
          Symphony of Chaos
        </h2>
        <p class="text-sm font-semibold text-electric-blue mt-2 tracking-widest uppercase">
          Synthesized by Christian de Graaf
        </p>

        <p class="text-base md:text-lg text-gray-300 leading-relaxed mt-6">
          An interactive, high-velocity particle-gravity simulation mapping mechanical orbit wells in vector space. Run your cursor across the canvas to distort the center of gravity and watch the cosmic particles morph in real-time.
        </p>

        <div class="mt-8 flex flex-wrap items-center gap-4">
          <div class="flex flex-col">
            <span class="text-[10px] uppercase font-bold text-gray-400 tracking-wider">Engine</span>
            <span class="text-sm font-bold text-white">Orbit Wells v2</span>
          </div>
          <div class="h-8 w-[1px] bg-white/20 mx-2"></div>
          <div class="flex flex-col">
            <span class="text-[10px] uppercase font-bold text-gray-400 tracking-wider">Particle Count</span>
            <span class="text-sm font-bold text-white">350 Vectors</span>
          </div>
          <div class="h-8 w-[1px] bg-white/20 mx-2"></div>
          <div class="flex flex-col">
            <span class="text-[10px] uppercase font-bold text-gray-400 tracking-wider">Status</span>
            <span class="text-sm font-bold text-emerald-400 flex items-center gap-1.5">
              <span class="w-1.5 h-1.5 rounded-full bg-emerald-400"></span> Live
            </span>
          </div>
        </div>

        <div class="mt-10 flex items-center space-x-4">
          <UButton
            size="lg"
            color="primary"
            class="rounded-full bg-electric-blue hover:bg-electric-blue/90 text-gray-950 font-bold shadow-lg shadow-electric-blue/30 px-6 py-3 transition-transform duration-300 hover:scale-105 active:scale-95"
          >
            Acquire Masterpiece
          </UButton>
          <button
            class="flex items-center space-x-2 text-sm font-bold text-white hover:text-electric-blue transition-colors duration-300"
            aria-label="Read more about Symphony of Chaos"
          >
            <span>Learn More</span>
            <UIcon name="i-lucide-arrow-right" class="w-4 h-4" />
          </button>
        </div>
      </div>
    </section>

    <!-- Philosophy / Manifesto Section -->
    <section
      id="about"
      class="relative py-28 px-6 md:px-12 max-w-4xl mx-auto text-center"
    >
      <div class="inline-flex items-center space-x-2 px-3 py-1 rounded-full border border-hot-pink/40 bg-hot-pink/10 mb-6">
        <UIcon name="i-lucide-award" class="w-4 h-4 text-hot-pink" />
        <span class="text-[10px] font-black tracking-widest text-hot-pink uppercase">Our Manifesto</span>
      </div>
      <h2 class="text-3xl md:text-5xl font-black text-gray-900 tracking-tight uppercase mb-8">
        Algorithms are the new brushstrokes.
      </h2>
      <p class="text-lg md:text-xl text-gray-700 leading-relaxed font-light mb-6">
        We believe that code is not merely a tool for productivity, but a highly expressive medium for sublime artistic creation.
      </p>
      <p class="text-base md:text-lg text-gray-600/90 leading-relaxed">
        CANVAS exists to blur the line between creator and observer. By synthesizing mathematical waveforms, fractal geometry, and chemical-like flows dynamically inside your browser, the artwork adapts, morphs, and responds to human existence. This isn't static media. It is a living, breathing artwork.
      </p>
    </section>

    <!-- Minimal Dark Footer -->
    <footer class="relative bg-brand-dark text-white py-16 px-6 md:px-12 border-t border-white/10">
      <div class="max-w-7xl mx-auto flex flex-col md:flex-row items-center justify-between gap-8">
        <div class="flex flex-col items-center md:items-start">
          <div class="flex items-center space-x-2.5 mb-3">
            <div class="w-6 h-6 rounded bg-gradient-to-tr from-coral to-hot-pink flex items-center justify-center font-bold text-[10px] text-white">
              C
            </div>
            <span class="text-lg font-bold tracking-widest">CANVAS</span>
          </div>
          <p class="text-xs text-gray-400 max-w-xs text-center md:text-left leading-relaxed">
            Where mathematical precision meets sublime aesthetic expression. Rendered dynamically.
          </p>
        </div>

        <!-- Links -->
        <div class="flex flex-wrap items-center justify-center gap-8 text-sm font-semibold text-gray-300">
          <a href="#hero" class="hover:text-coral transition-colors duration-300">Home</a>
          <a href="#gallery" class="hover:text-electric-blue transition-colors duration-300">Gallery</a>
          <a href="#featured" class="hover:text-hot-pink transition-colors duration-300">Featured</a>
          <a href="#about" class="hover:text-golden-yellow transition-colors duration-300">Philosophy</a>
        </div>

        <!-- Social Icons -->
        <div class="flex items-center space-x-4">
          <UButton
            to="https://github.com"
            target="_blank"
            icon="i-simple-icons-github"
            aria-label="GitHub"
            color="neutral"
            variant="ghost"
            class="text-gray-400 hover:text-white hover:bg-white/10"
          />
          <UButton
            to="https://twitter.com"
            target="_blank"
            icon="i-simple-icons-x"
            aria-label="X"
            color="neutral"
            variant="ghost"
            class="text-gray-400 hover:text-white hover:bg-white/10"
          />
          <UButton
            to="https://instagram.com"
            target="_blank"
            icon="i-simple-icons-instagram"
            aria-label="Instagram"
            color="neutral"
            variant="ghost"
            class="text-gray-400 hover:text-white hover:bg-white/10"
          />
        </div>
      </div>

      <div class="max-w-7xl mx-auto mt-12 pt-8 border-t border-white/5 flex flex-col sm:flex-row items-center justify-between gap-4 text-xs text-gray-500">
        <p>© {{ new Date().getFullYear() }} CANVAS Art Labs. All rights reserved.</p>
        <p>Built with Nuxt 4, Nuxt UI and Tailwind CSS.</p>
      </div>
    </footer>

  </div>
</template>

<style scoped>
/* Hero custom CSS overrides */
#hero h1 {
  background: linear-gradient(135deg, #ffffff 30%, rgba(255, 255, 255, 0.4) 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
</style>
