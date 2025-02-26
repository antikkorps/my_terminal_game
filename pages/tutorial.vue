<template>
  <div class="bg-gray-900 text-white">
    <div class="container mx-auto p-3 sm:p-6">
      <h1 class="text-xl sm:text-3xl font-bold mb-4 sm:mb-6 text-green-500">
        Tutoriel Terminal Linux
      </h1>

      <div class="grid md:grid-cols-3 gap-4 sm:gap-6">
        <!-- Navigation des leçons - d'abord en haut sur mobile -->
        <div class="bg-gray-800 p-3 sm:p-4 rounded-lg md:order-1 order-1">
          <div class="flex justify-between items-center mb-2 sm:mb-4">
            <h2 class="text-lg sm:text-xl font-bold">Leçons</h2>
            <button
              class="md:hidden bg-gray-700 px-2 py-1 rounded text-sm"
              @click="showLessonList = !showLessonList"
            >
              {{ showLessonList ? "Cacher" : "Voir toutes" }}
            </button>
          </div>

          <!-- Vue condensée sur mobile par défaut -->
          <div class="md:hidden" v-if="!showLessonList">
            <div class="flex justify-between items-center">
              <button
                @click="prevLesson"
                :disabled="currentLessonIndex === 0"
                class="bg-gray-700 px-3 py-1 rounded disabled:opacity-50"
              >
                ◄
              </button>
              <span class="text-green-400 font-bold"
                >{{ currentLessonIndex + 1 }}/{{ lessons.length }}</span
              >
              <button
                @click="nextLesson"
                :disabled="currentLessonIndex === lessons.length - 1"
                class="bg-gray-700 px-3 py-1 rounded disabled:opacity-50"
              >
                ►
              </button>
            </div>
          </div>

          <!-- Liste complète des leçons (toujours visible sur bureau, conditionnelle sur mobile) -->
          <ul
            v-if="showLessonList || window.innerWidth >= 768"
            class="space-y-1 sm:space-y-2 max-h-[40vh] md:max-h-[70vh] overflow-y-auto"
          >
            <li
              v-for="(lesson, index) in lessons"
              :key="index"
              @click="selectLesson(index)"
              :class="{ 'text-green-400 font-bold': currentLessonIndex === index }"
              class="cursor-pointer p-1.5 sm:p-2 hover:bg-gray-700 rounded text-sm sm:text-base"
            >
              <span class="mr-2">{{ index + 1 }}.</span>{{ lesson.title }}
            </li>
          </ul>
        </div>

        <!-- Contenu du tutoriel + terminal -->
        <div class="md:col-span-2 space-y-4 sm:space-y-6 md:order-2 order-2">
          <div class="bg-gray-800 p-4 sm:p-6 rounded-lg">
            <h2 class="text-xl sm:text-2xl font-bold mb-2 sm:mb-4">
              {{ currentLesson.title }}
            </h2>
            <div class="prose prose-invert prose-sm sm:prose mb-3 sm:mb-4">
              <p>{{ currentLesson.description }}</p>
              <div v-if="currentLesson.goals" class="mt-3 sm:mt-4">
                <h3 class="text-base sm:text-lg font-bold">Objectifs:</h3>
                <ul class="list-disc pl-4 sm:pl-6 text-sm sm:text-base">
                  <li v-for="(goal, idx) in currentLesson.goals" :key="idx" class="my-1">
                    {{ goal }}
                  </li>
                </ul>
              </div>
            </div>
          </div>

          <!-- Navigation de leçon pour mobile (sous le contenu) -->
          <div class="flex justify-between md:hidden">
            <button
              @click="prevLesson"
              :disabled="currentLessonIndex === 0"
              class="bg-gray-800 px-3 py-2 rounded disabled:opacity-50 text-sm"
            >
              ◄ Précédent
            </button>
            <button
              @click="nextLesson"
              :disabled="currentLessonIndex === lessons.length - 1"
              class="bg-green-700 px-3 py-2 rounded disabled:opacity-50 text-sm"
            >
              Suivant ►
            </button>
          </div>

          <!-- Terminal -->
          <TerminalEmulator
            :lesson="currentLesson"
            @lesson-completed="handleLessonCompleted"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, ref } from "vue"

const currentLessonIndex = ref(0)
const showLessonList = ref(false)
const window = ref({ innerWidth: 0 })

// Données des leçons
const lessons = [
  {
    title: "Introduction au terminal",
    description:
      "Bienvenue dans votre premier cours sur le terminal Linux ! Découvrons les bases.",
    goals: ["Comprendre ce qu'est un terminal", "Exécuter votre première commande"],
    validCommands: ["ls", "pwd", "clear"],
    successCondition: (history) => history.some((cmd) => cmd === "ls" || cmd === "pwd"),
  },
  {
    title: "Navigation dans les répertoires",
    description:
      "Apprenez à naviguer dans la structure des dossiers avec les commandes cd et ls.",
    goals: ["Se déplacer entre les dossiers", "Lister le contenu des dossiers"],
    validCommands: ["cd", "ls", "pwd", "clear"],
    successCondition: (history) => history.includes("cd") && history.includes("ls"),
  },
  // Autres leçons...
]

const currentLesson = computed(() => lessons[currentLessonIndex.value])

const selectLesson = (index) => {
  currentLessonIndex.value = index
  showLessonList.value = false // Cacher la liste sur mobile après sélection
}

const handleLessonCompleted = () => {
  // Passer à la leçon suivante si disponible
  if (currentLessonIndex.value < lessons.length - 1) {
    currentLessonIndex.value++
  }
}

const nextLesson = () => {
  if (currentLessonIndex.value < lessons.length - 1) {
    currentLessonIndex.value++
  }
}

const prevLesson = () => {
  if (currentLessonIndex.value > 0) {
    currentLessonIndex.value--
  }
}

onMounted(() => {
  // Déterminer la taille initiale de l'écran
  window.value.innerWidth =
    document.documentElement.clientWidth || document.body.clientWidth

  // Écouter les changements de taille d'écran
  const handleResize = () => {
    window.value.innerWidth =
      document.documentElement.clientWidth || document.body.clientWidth
  }

  globalThis.addEventListener("resize", handleResize)

  return () => {
    globalThis.removeEventListener("resize", handleResize)
  }
})
</script>
