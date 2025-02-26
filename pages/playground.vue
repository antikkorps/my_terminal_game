<template>
  <div class="min-h-screen bg-gray-900 text-white">
    <div class="container mx-auto p-6">
      <div class="flex justify-between items-center mb-6">
        <h1 class="text-3xl font-bold text-green-500">Mode Libre</h1>
        <NuxtLink to="/" class="text-green-400 hover:underline"
          >Retour à l'accueil</NuxtLink
        >
      </div>

      <div class="bg-gray-800 p-4 rounded-lg mb-6">
        <p>
          C'est votre environnement d'entraînement. Testez librement les commandes Linux.
          Aucune restriction, aucun objectif imposé. Expérimentez à votre rythme !
        </p>
      </div>

      <TerminalEmulator :freeMode="true" />
    </div>
  </div>
</template>

<script setup>
// Le terminal est en mode libre
</script>
