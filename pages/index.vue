<template>
  <div class="flex flex-col bg-gray-900 text-white">
    <!-- Hero section avec terminal animé -->
    <section
      class="flex-1 flex items-center justify-center p-6 py-20 relative overflow-hidden"
    >
      <div class="absolute inset-0 bg-grid opacity-10"></div>

      <div
        class="max-w-6xl mx-auto grid md:grid-cols-2 gap-12 items-center relative z-10"
      >
        <div class="text-left">
          <h1 class="text-5xl font-bold mb-6 text-green-500">
            <span class="block">Terminal</span>
            <span class="inline-block relative">
              Master
              <span class="absolute -right-4 top-0 animate-blink">_</span>
            </span>
          </h1>
          <p class="text-xl mb-8 text-gray-300">
            Maîtrisez le terminal Linux grâce à notre simulateur interactif. Pratiquez en
            toute sécurité et devenez un expert de la ligne de commande.
          </p>
          <div class="flex flex-wrap gap-4">
            <NuxtLink
              to="/tutorial"
              class="px-6 py-3 bg-gradient-to-r from-green-600 to-green-700 rounded-md hover:from-green-700 hover:to-green-800 transition transform hover:scale-105 shadow-lg"
            >
              Commencer l'aventure
            </NuxtLink>
            <NuxtLink
              to="/playground"
              class="px-6 py-3 bg-gray-700 bg-opacity-50 backdrop-blur-sm rounded-md border border-gray-600 hover:bg-gray-600 transition transform hover:scale-105"
            >
              Mode libre
            </NuxtLink>
          </div>
        </div>

        <!-- Terminal animé -->
        <div
          class="terminal-demo bg-black rounded-lg overflow-hidden border border-gray-700 shadow-2xl"
        >
          <div class="terminal-header bg-gray-800 p-2 flex justify-between items-center">
            <div class="flex space-x-2">
              <div class="w-3 h-3 bg-red-500 rounded-full"></div>
              <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
              <div class="w-3 h-3 bg-green-500 rounded-full"></div>
            </div>
            <div class="text-sm text-gray-400">bash</div>
          </div>
          <div class="p-4 font-mono text-sm h-64 overflow-hidden">
            <p class="text-green-400">
              user@linux:~$ <TypeWriter text="ls -la" :speed="80" />
            </p>
            <div v-show="animationStep > 0" class="mt-2 text-gray-300">
              <p>total 20</p>
              <p>drwxr-xr-x 5 user user 4096 Jul 15 10:24 .</p>
              <p>drwxr-xr-x 22 user user 4096 Jul 15 10:20 ..</p>
              <p>drwxr-xr-x 3 user user 4096 Jul 15 10:22 .config</p>
              <p>-rw-r--r-- 1 user user 220 Jul 15 10:18 .bash_profile</p>
              <p>drwxr-xr-x 2 user user 4096 Jul 15 10:21 Documents</p>
              <p>-rw-r--r-- 1 user user 56 Jul 15 10:23 notes.txt</p>
            </div>
            <p v-show="animationStep > 1" class="mt-2 text-green-400">
              user@linux:~$
              <TypeWriter
                text="cat notes.txt"
                :speed="80"
                :startDelay="2000"
                @typing-completed="animationStep = 3"
              />
            </p>
            <div v-show="animationStep > 2" class="mt-2 text-gray-300">
              <p>Bienvenue dans Terminal Master! Prêt à devenir un expert?</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Section des statistiques avec compteurs animés -->
    <section class="py-16 bg-gray-800">
      <div class="container mx-auto px-6">
        <h2 class="text-3xl font-bold mb-12 text-center">
          Pourquoi Apprendre le Terminal?
        </h2>

        <div class="grid md:grid-cols-4 gap-8 text-center">
          <div class="p-6 bg-gray-700 rounded-lg transform transition hover:scale-105">
            <div class="text-4xl font-bold text-green-400 mb-2">
              <CountUp :endValue="40" :duration="2" suffix="%" />
            </div>
            <p>Plus rapide que les interfaces graphiques</p>
          </div>

          <div class="p-6 bg-gray-700 rounded-lg transform transition hover:scale-105">
            <div class="text-4xl font-bold text-green-400 mb-2">
              <CountUp :endValue="100" :duration="2" suffix="+" />
            </div>
            <p>Commandes à découvrir</p>
          </div>

          <div class="p-6 bg-gray-700 rounded-lg transform transition hover:scale-105">
            <div class="text-4xl font-bold text-green-400 mb-2">
              <CountUp :endValue="15" :duration="2" />
            </div>
            <p>Leçons interactives</p>
          </div>

          <div class="p-6 bg-gray-700 rounded-lg transform transition hover:scale-105">
            <div class="text-4xl font-bold text-green-400 mb-2">
              <CountUp :endValue="24" :duration="2" suffix="/7" />
            </div>
            <p>Disponibilité</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Section des fonctionnalités avec animations au défilement -->
    <section class="py-16 bg-gray-900">
      <div class="container mx-auto px-6">
        <h2 class="text-3xl font-bold mb-16 text-center">Fonctionnalités</h2>

        <div class="grid md:grid-cols-3 gap-8">
          <div
            class="p-6 bg-gray-800 rounded-lg border border-gray-700 shadow-lg transform transition hover:-translate-y-2 hover:shadow-green-900/20"
          >
            <div class="text-green-400 text-3xl mb-4">
              <span class="iconify" data-icon="mdi:console-line"></span>
            </div>
            <h3 class="text-xl font-bold mb-3 text-green-400">Tutoriels interactifs</h3>
            <p class="text-gray-300">
              Apprenez étape par étape avec des exercices guidés qui renforcent vos
              compétences progressivement.
            </p>
          </div>

          <div
            class="p-6 bg-gray-800 rounded-lg border border-gray-700 shadow-lg transform transition hover:-translate-y-2 hover:shadow-green-900/20"
          >
            <div class="text-green-400 text-3xl mb-4">
              <span class="iconify" data-icon="mdi:keyboard"></span>
            </div>
            <h3 class="text-xl font-bold mb-3 text-green-400">Émulation réaliste</h3>
            <p class="text-gray-300">
              Utilisez un terminal qui reproduit fidèlement l'environnement Linux avec
              auto-complétion et historique des commandes.
            </p>
          </div>

          <div
            class="p-6 bg-gray-800 rounded-lg border border-gray-700 shadow-lg transform transition hover:-translate-y-2 hover:shadow-green-900/20"
          >
            <div class="text-green-400 text-3xl mb-4">
              <span class="iconify" data-icon="mdi:chart-line"></span>
            </div>
            <h3 class="text-xl font-bold mb-3 text-green-400">Progression structurée</h3>
            <p class="text-gray-300">
              Suivez votre progression et maîtrisez les commandes essentielles à votre
              rythme.
            </p>
          </div>
        </div>
      </div>
    </section>

    <!-- Témoignages en carrousel -->
    <section class="py-16 bg-gray-800">
      <div class="container mx-auto px-6">
        <h2 class="text-3xl font-bold mb-12 text-center">
          Ce que nos utilisateurs disent
        </h2>

        <div class="testimonial-carousel relative">
          <div class="overflow-hidden">
            <div
              class="flex transition-transform duration-500"
              :style="`transform: translateX(-${activeTestimonial * 100}%)`"
            >
              <div
                v-for="(testimonial, index) in testimonials"
                :key="index"
                class="min-w-full px-4"
              >
                <div class="bg-gray-700 p-8 rounded-lg">
                  <div class="flex items-center mb-4">
                    <div
                      class="w-12 h-12 rounded-full bg-green-800 flex items-center justify-center text-xl font-bold text-white"
                    >
                      {{ testimonial.name.charAt(0) }}
                    </div>
                    <div class="ml-4">
                      <h4 class="font-bold">{{ testimonial.name }}</h4>
                      <p class="text-sm text-gray-400">{{ testimonial.role }}</p>
                    </div>
                  </div>
                  <div class="mb-4 text-yellow-400">★★★★★</div>
                  <p class="text-gray-300 italic">"{{ testimonial.text }}"</p>
                </div>
              </div>
            </div>
          </div>

          <div class="flex justify-center mt-6 space-x-2">
            <button
              v-for="(_, index) in testimonials"
              :key="index"
              @click="activeTestimonial = index"
              :class="[
                'w-3 h-3 rounded-full',
                activeTestimonial === index ? 'bg-green-500' : 'bg-gray-600',
              ]"
            ></button>
          </div>
        </div>
      </div>
    </section>

    <!-- Call to action -->
    <section class="py-16 bg-gradient-to-b from-gray-900 to-green-900">
      <div class="container mx-auto px-6 text-center">
        <h2 class="text-3xl font-bold mb-6">Prêt à maîtriser le terminal?</h2>
        <p class="text-xl mb-8 max-w-2xl mx-auto">
          Rejoignez des milliers d'apprenants et commencez votre voyage vers la maîtrise
          du terminal Linux dès aujourd'hui.
        </p>
        <NuxtLink
          to="/tutorial"
          class="px-8 py-4 bg-green-500 text-lg rounded-md hover:bg-green-600 transition transform hover:scale-105 inline-block"
        >
          Démarrer maintenant →
        </NuxtLink>
      </div>
    </section>

    <!-- Footer retiré car déplacé dans le layout -->
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue"

// Animation du terminal
const animationStep = ref(0)

// Pour le carrousel de témoignages
const activeTestimonial = ref(0)
const testimonials = [
  {
    name: "Marie L.",
    role: "Développeuse Web",
    text: "Terminal Master m'a permis de surmonter ma peur de la ligne de commande. Je suis maintenant beaucoup plus productive dans mon travail!",
  },
  {
    name: "Thomas D.",
    role: "Étudiant en informatique",
    text: "Un excellent outil pédagogique. Les leçons sont progressives et l'interface reproduit parfaitement l'expérience d'un vrai terminal.",
  },
  {
    name: "Sophie M.",
    role: "DevOps Engineer",
    text: "Même en tant que professionnelle, j'ai appris quelques astuces que je ne connaissais pas. Je recommande à tous les niveaux!",
  },
]

// Rotation automatique du carrousel
onMounted(() => {
  // Démarrer l'animation du terminal
  setTimeout(() => {
    animationStep.value = 1
    setTimeout(() => {
      animationStep.value = 2
    }, 2000)
  }, 1500)

  // Carrousel
  const interval = setInterval(() => {
    activeTestimonial.value = (activeTestimonial.value + 1) % testimonials.length
  }, 5000)

  return () => clearInterval(interval)
})
</script>

<style scoped>
.bg-grid {
  background-image: linear-gradient(to right, rgba(55, 65, 81, 0.3) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(55, 65, 81, 0.3) 1px, transparent 1px);
  background-size: 20px 20px;
}

@keyframes blink {
  0%,
  100% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
}

.animate-blink {
  animation: blink 1s infinite;
}
</style>
