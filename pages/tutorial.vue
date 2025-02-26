<template>
  <div class="bg-gray-900 text-white">
    <div class="container mx-auto p-6">
      <h1 class="text-3xl font-bold mb-6 text-green-500">Tutoriel Terminal Linux</h1>

      <div class="grid md:grid-cols-3 gap-6">
        <!-- Navigation des leçons -->
        <div class="bg-gray-800 p-4 rounded-lg">
          <h2 class="text-xl font-bold mb-4">Leçons</h2>
          <ul class="space-y-2">
            <li
              v-for="(lesson, index) in lessons"
              :key="index"
              @click="selectLesson(index)"
              :class="{ 'text-green-400 font-bold': currentLessonIndex === index }"
              class="cursor-pointer p-2 hover:bg-gray-700 rounded"
            >
              {{ lesson.title }}
            </li>
          </ul>
        </div>

        <!-- Contenu du tutoriel + terminal -->
        <div class="md:col-span-2 space-y-6">
          <div class="bg-gray-800 p-6 rounded-lg">
            <h2 class="text-2xl font-bold mb-4">{{ currentLesson.title }}</h2>
            <div class="prose prose-invert mb-4">
              <p>{{ currentLesson.description }}</p>
              <div v-if="currentLesson.goals" class="mt-4">
                <h3 class="text-lg font-bold">Objectifs:</h3>
                <ul class="list-disc pl-6">
                  <li v-for="(goal, idx) in currentLesson.goals" :key="idx">
                    {{ goal }}
                  </li>
                </ul>
              </div>
            </div>
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
import { computed, ref } from "vue"

const currentLessonIndex = ref(0)

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
}

const handleLessonCompleted = () => {
  // Passer à la leçon suivante si disponible
  if (currentLessonIndex.value < lessons.length - 1) {
    currentLessonIndex.value++
  }
}
</script>
