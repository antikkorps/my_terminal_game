<template>
  <div
    class="terminal bg-white border border-gray-300 p-6 rounded-lg shadow-md w-full max-w-lg"
  >
    <div class="output bg-gray-100 p-4 rounded mb-4">{{ output }}</div>
    <CommandInput @command="handleCommand" />
  </div>
</template>

<script setup>
import { ref } from "vue"
import CommandInput from "./CommandInput.vue"

const output = ref("")

const handleCommand = (command) => {
  // Logique pour traiter les commandes
  output.value = `Résultat de la commande : ${command}`
}
</script>
