<script setup>
import { ref, computed } from 'vue'
import Nes from './components/Nes.vue'
import Sega from './components/Sega.vue'
import Home from './components/Home.vue'

const routes = {
  '/': Home,
  '/nes': Nes,
  '/sega': Sega
}

const currentPath = ref(window.location.hash)

window.addEventListener('hashchange', () => {
  currentPath.value = window.location.hash
})

const currentView = computed(() => {
  return routes[currentPath.value.slice(1) || '/']
})
</script>

<template>
  <component :is="currentView" />
</template>