<template>
  <div class="min-h-screen bg-gray-900 text-white py-10">
    <div class="container mx-auto px-4">
      <div class="max-w-4xl mx-auto">
        <!-- Header -->
        <div class="text-center mb-8">
          <h1 class="text-3xl sm:text-4xl md:text-5xl font-bold mb-4 text-green-500">
            Terminal Master: L'Aventure
          </h1>
          <p class="text-lg text-gray-300 max-w-2xl mx-auto">
            Choisissez votre aventure et mettez vos compétences en ligne de commande à
            l'épreuve dans des missions palpitantes.
          </p>
        </div>

        <!-- Mode sélection -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-10">
          <div
            @click="selectMode('adventure')"
            class="bg-gray-800 rounded-lg overflow-hidden border border-gray-700 hover:border-green-500 transform hover:-translate-y-1 transition cursor-pointer"
          >
            <div class="p-6">
              <h2 class="text-xl font-bold text-green-400 mb-2">Mode Histoire</h2>
              <p class="text-gray-300 mb-4">
                Suivez une aventure palpitante à travers une série de missions narratives.
                Progressez d'une mission à l'autre en résolvant des défis liés aux
                commandes Linux.
              </p>
              <div class="flex justify-between items-center">
                <span class="bg-green-900 text-green-200 rounded px-2 py-1 text-xs"
                  >Pour tous niveaux</span
                >
                <span class="text-gray-400 text-sm">15+ missions</span>
              </div>
            </div>
          </div>

          <div
            @click="selectMode('sandbox')"
            class="bg-gray-800 rounded-lg overflow-hidden border border-gray-700 hover:border-blue-500 transform hover:-translate-y-1 transition cursor-pointer"
          >
            <div class="p-6">
              <h2 class="text-xl font-bold text-blue-400 mb-2">Mode Libre</h2>
              <p class="text-gray-300 mb-4">
                Explorez librement un environnement Linux simulé. Idéal pour pratiquer les
                commandes sans contraintes ou pour tester différentes approches.
              </p>
              <div class="flex justify-between items-center">
                <span class="bg-blue-900 text-blue-200 rounded px-2 py-1 text-xs"
                  >Mode expérimentation</span
                >
                <span class="text-gray-400 text-sm">Toutes commandes</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Défis récents -->
        <div class="bg-gray-800 rounded-lg p-6">
          <h2 class="text-xl font-bold mb-4 text-center">Défis hebdomadaires</h2>

          <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
            <div class="bg-gray-900 p-4 rounded-lg border border-gray-700">
              <h3 class="text-yellow-400 font-bold">Le Labyrinthe des Fichiers</h3>
              <p class="text-sm text-gray-300 mb-2">
                Naviguez dans un dédale de dossiers pour trouver un fichier caché.
              </p>
              <div class="flex justify-between">
                <span class="text-xs text-gray-500">Difficulté: Moyenne</span>
                <button class="text-xs bg-yellow-800 text-yellow-200 px-2 py-1 rounded">
                  Essayer
                </button>
              </div>
            </div>

            <div class="bg-gray-900 p-4 rounded-lg border border-gray-700">
              <h3 class="text-red-400 font-bold">Contre la montre</h3>
              <p class="text-sm text-gray-300 mb-2">
                Exécutez une série de commandes avant la fin du compte à rebours!
              </p>
              <div class="flex justify-between">
                <span class="text-xs text-gray-500">Difficulté: Difficile</span>
                <button class="text-xs bg-red-800 text-red-200 px-2 py-1 rounded">
                  Essayer
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { useRouter } from "vue-router"

const router = useRouter()

const selectMode = (mode) => {
  if (mode === "adventure") {
    router.push("/adventure")
  } else if (mode === "sandbox") {
    router.push("/playground")
  }
}
</script>
