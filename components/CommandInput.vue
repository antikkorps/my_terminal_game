<template>
  <form @submit.prevent="handleSubmit" class="flex">
    <input
      type="text"
      v-model="input"
      placeholder="Entrez une commande"
      class="flex-grow border border-gray-300 p-2 rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
    />
    <button
      type="submit"
      class="bg-blue-500 text-white p-2 rounded-r hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500"
    >
      Exécuter
    </button>
  </form>
</template>

<script setup>
import { ref } from "vue"

const input = ref("")

const emit = defineEmits(["command"])

const handleSubmit = () => {
  emit("command", input.value)
  input.value = ""
}
</script>
