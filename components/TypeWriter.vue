<template>
  <span>{{ displayedText }}<span v-if="isTyping" class="cursor">_</span></span>
</template>

<script setup>
import { onMounted, ref, watch } from "vue"

const props = defineProps({
  text: {
    type: String,
    required: true,
  },
  speed: {
    type: Number,
    default: 100,
  },
  startDelay: {
    type: Number,
    default: 0,
  },
})

const emit = defineEmits(["typing-completed"])

const displayedText = ref("")
const isTyping = ref(false)
let currentIndex = 0
let typingInterval

const typeCharacter = () => {
  if (currentIndex < props.text.length) {
    isTyping.value = true
    displayedText.value += props.text.charAt(currentIndex)
    currentIndex++
  } else {
    isTyping.value = false
    clearInterval(typingInterval)
    emit("typing-completed")
  }
}

const startTyping = () => {
  currentIndex = 0
  displayedText.value = ""
  isTyping.value = true

  setTimeout(() => {
    typingInterval = setInterval(typeCharacter, props.speed)
  }, props.startDelay)
}

onMounted(() => {
  startTyping()
})

// Redémarrer l'animation si le texte change
watch(
  () => props.text,
  () => {
    clearInterval(typingInterval)
    startTyping()
  }
)
</script>

<style scoped>
.cursor {
  animation: blink 1s infinite;
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
</style>
