<template>
  <span ref="countElement">{{ displayValue }}{{ suffix }}</span>
</template>

<script setup>
import { onMounted, onUnmounted, ref, watch } from "vue"

const props = defineProps({
  endValue: {
    type: Number,
    required: true,
  },
  duration: {
    type: Number,
    default: 2, // seconds
  },
  suffix: {
    type: String,
    default: "",
  },
})

const countElement = ref(null)
const displayValue = ref(0)
let animationId
let observer = null

const animate = (timestamp, startValue, endValue, startTime, duration) => {
  if (!startTime) {
    startTime = timestamp
  }

  const elapsed = timestamp - startTime
  const progress = Math.min(elapsed / (duration * 1000), 1)

  displayValue.value = Math.floor(startValue + progress * (endValue - startValue))

  if (progress < 1) {
    animationId = requestAnimationFrame((newTimestamp) =>
      animate(newTimestamp, startValue, endValue, startTime, duration)
    )
  }
}

const startAnimation = () => {
  const startValue = 0
  const endValue = props.endValue

  cancelAnimationFrame(animationId)
  animationId = requestAnimationFrame((timestamp) =>
    animate(timestamp, startValue, endValue, null, props.duration)
  )
}

onMounted(() => {
  // Vérifier si IntersectionObserver est disponible (certains navigateurs mobiles anciens ne le supportent pas)
  if ("IntersectionObserver" in window) {
    observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            startAnimation()
            // Déconnecter l'observer après avoir déclenché l'animation
            if (observer) {
              observer.disconnect()
            }
          }
        })
      },
      { threshold: 0.1 }
    )

    // Observer l'élément actuel
    if (countElement.value) {
      observer.observe(countElement.value)
    }
  } else {
    // Fallback pour les navigateurs qui ne supportent pas IntersectionObserver
    startAnimation()
  }
})

onUnmounted(() => {
  // Nettoyer l'animation et l'observer
  cancelAnimationFrame(animationId)

  if (observer) {
    observer.disconnect()
    observer = null
  }
})

// Redémarrer l'animation si la valeur finale change
watch(
  () => props.endValue,
  () => {
    startAnimation()
  }
)
</script>
