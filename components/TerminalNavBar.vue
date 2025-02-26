<template>
  <div class="terminal-navbar bg-black border-b border-green-500 sticky top-0 z-50">
    <div class="container mx-auto px-3">
      <!-- Barre de titre du terminal -->
      <div class="bg-gray-800 flex items-center justify-between px-2 py-1">
        <div class="flex space-x-2 items-center">
          <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-red-500 rounded-full"></div>
          <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-yellow-500 rounded-full"></div>
          <div class="w-2.5 h-2.5 sm:w-3 sm:h-3 bg-green-500 rounded-full"></div>
        </div>
        <div
          class="text-xs sm:text-sm text-gray-400 truncate max-w-[150px] sm:max-w-none"
        >
          terminal-master@{{ currentPath }}
        </div>
        <div class="text-xs sm:text-sm text-gray-400">{{ formattedTime }}</div>
      </div>

      <!-- Contenu de la navbar -->
      <div class="p-2 font-mono text-sm">
        <div class="flex items-center justify-between">
          <!-- Logo/Titre comme prompt terminal - adapté pour mobile -->
          <NuxtLink
            to="/"
            class="text-green-400 font-bold flex items-center hover:underline text-xs sm:text-sm whitespace-nowrap"
          >
            <span class="hidden sm:inline">root@</span><span>terminal-master:</span
            ><span class="text-blue-400">~</span><span class="text-white">#</span>
            <span class="text-green-500 ml-1 sm:ml-2 hidden xs:inline">./t-master</span>
            <span class="text-green-500 ml-1 sm:ml-2 xs:hidden">./tm</span>
          </NuxtLink>

          <!-- Bouton menu mobile -->
          <button
            @click="isMenuOpen = !isMenuOpen"
            class="px-2 py-1 text-green-400 sm:hidden bg-gray-800 rounded text-xs"
          >
            <span v-if="!isMenuOpen">$ menu</span>
            <span v-else>$ exit</span>
          </button>

          <!-- Navigation principale - cachée sur mobile -->
          <div
            class="hidden sm:flex flex-1 justify-center space-x-1 lg:space-x-4 px-2 text-white overflow-x-auto py-1 terminal-tabs"
          >
            <NuxtLink
              v-for="(item, index) in menuItems"
              :key="index"
              :to="item.path"
              class="terminal-tab px-2 py-1 rounded-t transition-colors duration-150 text-xs lg:text-sm whitespace-nowrap"
              :class="
                isActive(item.path) ? 'bg-green-800 text-white' : 'hover:bg-gray-800'
              "
            >
              <span class="text-gray-400">$</span> {{ item.command }}
            </NuxtLink>
          </div>
        </div>

        <!-- Menu mobile -->
        <div
          v-if="isMenuOpen"
          class="sm:hidden py-2 px-1 bg-gray-900 mt-2 rounded border border-gray-700 text-xs"
        >
          <NuxtLink
            v-for="(item, index) in menuItems"
            :key="index"
            :to="item.path"
            class="block py-2 px-2 hover:bg-gray-800 text-white"
            :class="isActive(item.path) ? 'bg-green-900' : ''"
            @click="isMenuOpen = false"
          >
            <span class="text-gray-400">$</span> {{ item.command }}
            <span class="text-gray-400 text-xs ml-1">- {{ item.description }}</span>
          </NuxtLink>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue"
import { useRoute } from "vue-router"

const route = useRoute()
const isMenuOpen = ref(false)
const currentTime = ref(new Date())
const currentPath = ref("home")

// Menu items avec commandes terminal
const menuItems = [
  { path: "/", command: "home", description: "Page d'accueil" },
  { path: "/tutorial", command: "learn", description: "Tutoriels interactifs" },
  { path: "/playground", command: "play", description: "Mode libre" },
  { path: "/about", command: "about", description: "À propos" },
  { path: "/contact", command: "contact", description: "Nous contacter" },
]

// Vérifier si un lien est actif
const isActive = (path) => {
  return route.path === path
}

// Mise à jour du chemin courant dans le style terminal
const updateCurrentPath = () => {
  if (route.path === "/") {
    currentPath.value = "home"
  } else {
    currentPath.value = route.path.substring(1)
  }
}

// Format de l'heure en mode terminal
const formattedTime = computed(() => {
  const hours = currentTime.value.getHours().toString().padStart(2, "0")
  const minutes = currentTime.value.getMinutes().toString().padStart(2, "0")
  const seconds = currentTime.value.getSeconds().toString().padStart(2, "0")
  return `${hours}:${minutes}:${seconds}`
})

// Mise à jour de l'heure chaque seconde
let intervalId
onMounted(() => {
  updateCurrentPath()
  intervalId = setInterval(() => {
    currentTime.value = new Date()
  }, 1000)
})

onUnmounted(() => {
  clearInterval(intervalId)
})
</script>

<style scoped>
.terminal-navbar {
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.terminal-tabs {
  scrollbar-width: thin;
  scrollbar-color: #4b5563 #1f2937;
}

.terminal-tabs::-webkit-scrollbar {
  height: 4px;
}

.terminal-tabs::-webkit-scrollbar-track {
  background: #1f2937;
}

.terminal-tabs::-webkit-scrollbar-thumb {
  background-color: #4b5563;
  border-radius: 3px;
}

.terminal-tab {
  position: relative;
  overflow: hidden;
  white-space: nowrap;
}

.terminal-tab::before {
  content: "";
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #10b981, transparent);
  transform: translateX(-100%);
  transition: transform 0.3s;
}

.terminal-tab:hover::before {
  transform: translateX(0);
}

@keyframes cursor {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

/* Ajout d'un breakpoint personnalisé pour les très petits écrans */
@media (min-width: 400px) {
  .xs\:hidden {
    display: none;
  }
  .xs\:inline {
    display: inline;
  }
}

@media (max-width: 399px) {
  .xs\:hidden {
    display: inline;
  }
  .xs\:inline {
    display: none;
  }
}
</style>
