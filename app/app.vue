<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

useHead({
  title: 'CANVAS - Generative Art Gallery',
  meta: [
    { name: 'viewport', content: 'width=device-width, initial-scale=1' },
    { name: 'description', content: 'CANVAS is a living generative art gallery rendering computational beauty in real-time. Experience flowing gradients, interactive particles, and procedurally synthesized masterpieces.' }
  ],
  link: [
    { rel: 'icon', href: '/favicon.ico' },
    { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
    { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: 'anonymous' },
    { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;700&display=swap' }
  ],
  htmlAttrs: {
    lang: 'en'
  }
})

const isScrolled = ref(false)
const isMobileMenuOpen = ref(false)

watch(isMobileMenuOpen, (isOpen) => {
  if (typeof window !== 'undefined') {
    document.body.style.overflow = isOpen ? 'hidden' : ''
  }
})

onMounted(() => {
  window.addEventListener('scroll', () => {
    isScrolled.value = window.scrollY > 50
  })
})
</script>

<template>
  <UApp>
    <!-- Live canvas rendering the flowing ambient art background globally -->
    <LivingBackground />

    <!-- Expressive Glassmorphic Navigation Bar -->
    <header
      class="fixed top-0 left-0 right-0 z-50 transition-all duration-500 ease-out py-4 px-6 md:px-12"
      :class="[
        isScrolled 
          ? 'bg-white/45 backdrop-blur-xl border-b border-white/30 py-3 shadow-lg shadow-black/5' 
          : 'bg-transparent border-b border-white/0 py-5'
      ]"
    >
      <div class="max-w-7xl mx-auto flex items-center justify-between">
        <!-- Site Logo -->
        <NuxtLink to="/" class="flex items-center space-x-2.5 group">
          <div class="relative w-8 h-8 rounded-lg bg-gradient-to-tr from-coral via-hot-pink to-electric-blue p-[1.5px] shadow-sm transition-transform duration-500 group-hover:rotate-12">
            <div class="w-full h-full rounded-[7px] bg-white flex items-center justify-center font-bold text-xs tracking-tighter text-gray-900 group-hover:scale-95 transition-all">
              C
            </div>
          </div>
          <span class="text-xl font-bold tracking-widest text-gray-900 group-hover:text-coral transition-colors duration-300">
            CANVAS
          </span>
        </NuxtLink>

        <!-- Navigation Links -->
        <nav class="hidden md:flex items-center space-x-8">
          <a
            href="#hero"
            class="text-sm font-semibold text-gray-700 hover:text-coral transition-colors duration-300 relative after:absolute after:bottom-[-4px] after:left-0 after:h-[2px] after:w-0 hover:after:w-full after:bg-coral after:transition-all after:duration-300"
          >
            Home
          </a>
          <a
            href="#gallery"
            class="text-sm font-semibold text-gray-700 hover:text-electric-blue transition-colors duration-300 relative after:absolute after:bottom-[-4px] after:left-0 after:h-[2px] after:w-0 hover:after:w-full after:bg-electric-blue after:transition-all after:duration-300"
          >
            Gallery
          </a>
          <a
            href="#featured"
            class="text-sm font-semibold text-gray-700 hover:text-hot-pink transition-colors duration-300 relative after:absolute after:bottom-[-4px] after:left-0 after:h-[2px] after:w-0 hover:after:w-full after:bg-hot-pink after:transition-all after:duration-300"
          >
            Featured
          </a>
          <a
            href="#about"
            class="text-sm font-semibold text-gray-700 hover:text-golden-yellow transition-colors duration-300 relative after:absolute after:bottom-[-4px] after:left-0 after:h-[2px] after:w-0 hover:after:w-full after:bg-golden-yellow after:transition-all after:duration-300"
          >
            Philosophy
          </a>
        </nav>

        <!-- Actions -->
        <div class="flex items-center space-x-3.5">
          <UButton
            to="#gallery"
            size="md"
            color="neutral"
            variant="subtle"
            class="hidden sm:inline-flex rounded-full bg-white/50 border border-white/60 hover:bg-white hover:border-coral/50 transition-all duration-300 text-gray-800 font-bold"
          >
            Explore Live
          </UButton>

          <!-- Hamburger Menu Button (Mobile only) -->
          <button
            @click="isMobileMenuOpen = !isMobileMenuOpen"
            class="flex md:hidden relative w-10 h-10 rounded-full bg-white/50 border border-white/60 hover:bg-white items-center justify-center text-gray-800 hover:text-black transition-all duration-300 focus:outline-none"
            aria-label="Toggle mobile menu"
          >
            <div class="relative w-5 h-3.5 flex flex-col justify-between">
              <span 
                class="w-5 h-0.5 bg-current rounded transition-all duration-300 transform origin-left"
                :class="{ 'rotate-45 translate-x-[2px] translate-y-[-1px]': isMobileMenuOpen }"
              ></span>
              <span 
                class="w-5 h-0.5 bg-current rounded transition-all duration-300"
                :class="{ 'opacity-0 scale-x-0': isMobileMenuOpen }"
              ></span>
              <span 
                class="w-5 h-0.5 bg-current rounded transition-all duration-300 transform origin-left"
                :class="{ '-rotate-45 translate-x-[2px] translate-y-[1px]': isMobileMenuOpen }"
              ></span>
            </div>
          </button>
        </div>
      </div>
    </header>

    <!-- Mobile Navigation Drawer Overlay -->
    <Transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 translate-y-[-10px]"
      enter-to-class="opacity-100 translate-y-0"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 translate-y-0"
      leave-to-class="opacity-0 translate-y-[-10px]"
    >
      <div
        v-if="isMobileMenuOpen"
        class="fixed inset-0 pt-24 pb-8 px-6 z-40 md:hidden bg-white/95 backdrop-blur-2xl flex flex-col justify-between"
      >
        <!-- Nav Links -->
        <nav class="flex flex-col space-y-6 text-center mt-4">
          <a
            href="#hero"
            @click="isMobileMenuOpen = false"
            class="text-2xl font-bold text-gray-900 hover:text-coral transition-colors duration-300 py-2 border-b border-gray-100"
          >
            Home
          </a>
          <a
            href="#gallery"
            @click="isMobileMenuOpen = false"
            class="text-2xl font-bold text-gray-900 hover:text-electric-blue transition-colors duration-300 py-2 border-b border-gray-100"
          >
            Gallery
          </a>
          <a
            href="#featured"
            @click="isMobileMenuOpen = false"
            class="text-2xl font-bold text-gray-900 hover:text-hot-pink transition-colors duration-300 py-2 border-b border-gray-100"
          >
            Featured
          </a>
          <a
            href="#about"
            @click="isMobileMenuOpen = false"
            class="text-2xl font-bold text-gray-900 hover:text-golden-yellow transition-colors duration-300 py-2 border-b border-gray-100"
          >
            Philosophy
          </a>
        </nav>

        <!-- Mobile Drawer Actions / Footer info -->
        <div class="flex flex-col items-center space-y-6 pb-8">
          <UButton
            to="#gallery"
            @click="isMobileMenuOpen = false"
            size="lg"
            color="primary"
            class="w-full rounded-full bg-coral hover:bg-coral/90 text-white font-bold shadow-lg shadow-coral/30 py-3.5 text-center flex justify-center"
          >
            Explore Live Gallery
          </UButton>
          <div class="flex items-center space-x-6">
            <a href="https://github.com" target="_blank" aria-label="GitHub" class="text-gray-500 hover:text-gray-900 transition-colors">
              <UIcon name="i-simple-icons-github" class="w-6 h-6" />
            </a>
            <a href="https://twitter.com" target="_blank" aria-label="X" class="text-gray-500 hover:text-gray-900 transition-colors">
              <UIcon name="i-simple-icons-x" class="w-6 h-6" />
            </a>
            <a href="https://instagram.com" target="_blank" aria-label="Instagram" class="text-gray-500 hover:text-gray-900 transition-colors">
              <UIcon name="i-simple-icons-instagram" class="w-6 h-6" />
            </a>
          </div>
        </div>
      </div>
    </Transition>

    <!-- Main App Container -->
    <main class="relative z-10 w-full min-h-screen">
      <NuxtPage />
    </main>
  </UApp>
</template>

<style>
/* Nuxt App Overrides and Global resets */
.u-app {
  background: transparent !important;
}
</style>
