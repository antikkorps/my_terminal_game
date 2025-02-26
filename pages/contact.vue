<template>
  <div class="container mx-auto p-6 py-12">
    <div class="max-w-3xl mx-auto">
      <div class="bg-black border border-gray-700 rounded-lg overflow-hidden">
        <div class="terminal-header bg-gray-800 p-2 flex justify-between items-center">
          <div class="flex space-x-2">
            <div class="w-3 h-3 bg-red-500 rounded-full"></div>
            <div class="w-3 h-3 bg-yellow-500 rounded-full"></div>
            <div class="w-3 h-3 bg-green-500 rounded-full"></div>
          </div>
          <div class="text-sm text-gray-400">contact.sh</div>
        </div>

        <div class="p-6 font-mono">
          <div class="mb-6">
            <p class="text-green-400">root@terminal-master:~$ ./contact.sh</p>
            <p class="text-white mt-2">
              Bienvenue dans le formulaire de contact de Terminal Master
            </p>
            <p class="text-gray-400 mt-1">Veuillez remplir les informations suivantes:</p>
          </div>

          <form @submit.prevent="submitForm" class="space-y-4">
            <div>
              <p class="text-green-400">user@terminal-master:~$ read NAME</p>
              <div class="flex items-center">
                <span class="text-yellow-400 mr-2">Nom:</span>
                <input
                  v-model="formData.name"
                  type="text"
                  class="flex-1 bg-gray-900 border border-gray-700 text-white p-2 rounded focus:outline-none focus:border-green-500"
                  required
                />
              </div>
            </div>

            <div>
              <p class="text-green-400">user@terminal-master:~$ read EMAIL</p>
              <div class="flex items-center">
                <span class="text-yellow-400 mr-2">Email:</span>
                <input
                  v-model="formData.email"
                  type="email"
                  class="flex-1 bg-gray-900 border border-gray-700 text-white p-2 rounded focus:outline-none focus:border-green-500"
                  required
                />
              </div>
            </div>

            <div>
              <p class="text-green-400">
                user@terminal-master:~$ read -p "Message" MESSAGE
              </p>
              <div>
                <span class="text-yellow-400 block mb-1">Message:</span>
                <textarea
                  v-model="formData.message"
                  rows="4"
                  class="w-full bg-gray-900 border border-gray-700 text-white p-2 rounded focus:outline-none focus:border-green-500"
                  required
                ></textarea>
              </div>
            </div>

            <div>
              <button
                type="submit"
                class="w-full bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded transition duration-200 flex items-center justify-center"
                :class="{ 'opacity-50': isSubmitting }"
                :disabled="isSubmitting"
              >
                <span class="mr-2">Envoyer</span>
                <span class="text-xs">[ENTER]</span>
              </button>
            </div>
          </form>

          <div v-if="formSubmitted" class="mt-6 p-4 bg-gray-800 rounded">
            <p class="text-green-400">Message envoyé avec succès!</p>
            <p class="text-gray-300 mt-2">
              Nous vous répondrons dans les plus brefs délais.
            </p>
            <p class="text-green-400 mt-2">
              root@terminal-master:~$<span class="animate-blink ml-1">_</span>
            </p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue"

const formData = ref({
  name: "",
  email: "",
  message: "",
})

const isSubmitting = ref(false)
const formSubmitted = ref(false)

const submitForm = async () => {
  isSubmitting.value = true

  // Simuler un délai réseau
  await new Promise((resolve) => setTimeout(resolve, 1000))

  // Dans une application réelle, vous enverriez les données à un serveur ici
  console.log("Form data:", formData.value)

  formSubmitted.value = true
  isSubmitting.value = false

  // Réinitialiser le formulaire
  formData.value = {
    name: "",
    email: "",
    message: "",
  }
}
</script>

<style scoped>
.animate-blink {
  animation: blink 1s step-end infinite;
}

@keyframes blink {
  50% {
    opacity: 0;
  }
}
</style>
