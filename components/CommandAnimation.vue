<template>
  <div class="command-animation">
    <span class="prefix">{{ prefix }}</span>
    <span class="command">{{ displayedCommand }}</span>
    <span v-if="showCursor" class="cursor">_</span>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref } from "vue"

const props = defineProps({
  commands: {
    type: Array,
    default: () => [
      "ls",
      "cd ~/.config",
      "vim .bashrc",
      "cat /etc/passwd",
      "find / -name *.txt",
    ],
  },
  prefix: {
    type: String,
    default: "$ ",
  },
  typingSpeed: {
    type: Number,
    default: 100,
  },
  pauseBetween: {
    type: Number,
    default: 2000,
  },
})

const currentCommandIndex = ref(0)
const displayedCommand = ref("")
const charIndex = ref(0)
const showCursor = ref(true)
let typingTimer = null
let cursorTimer = null

const currentCommand = computed(() => {
  return props.commands[currentCommandIndex.value % props.commands.length]
})

const typeNextChar = () => {
  if (charIndex.value < currentCommand.value.length) {
    displayedCommand.value = currentCommand.value.substring(0, charIndex.value + 1)
    charIndex.value++
    typingTimer = setTimeout(typeNextChar, props.typingSpeed)
  } else {
    // Pause at the end of command, then move to next command
    typingTimer = setTimeout(() => {
      // Clear current command gradually
      clearCommand()
    }, props.pauseBetween)
  }
}

const clearCommand = () => {
  if (displayedCommand.value.length > 0) {
    displayedCommand.value = displayedCommand.value.slice(0, -1)
    typingTimer = setTimeout(clearCommand, props.typingSpeed / 2)
  } else {
    currentCommandIndex.value++
    charIndex.value = 0
    typingTimer = setTimeout(typeNextChar, props.typingSpeed * 3)
  }
}

const blink = () => {
  showCursor.value = !showCursor.value
}

onMounted(() => {
  typingTimer = setTimeout(typeNextChar, props.typingSpeed * 3)
  cursorTimer = setInterval(blink, 500)
})

onUnmounted(() => {
  clearTimeout(typingTimer)
  clearInterval(cursorTimer)
})
</script>

<style scoped>
.command-animation {
  font-family: monospace;
  white-space: nowrap;
  overflow: hidden;
}

.prefix {
  color: #10b981; /* green-500 */
}

.command {
  color: white;
}

.cursor {
  color: white;
  font-weight: bold;
}
</style>
