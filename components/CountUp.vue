<template>
  <span>{{ displayValue }}{{ suffix }}</span>
</template>

<script setup>
import { onMounted, ref, watch } from "vue"

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

const displayValue = ref(0)
let animationId

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
  // Utiliser IntersectionObserver pour démarrer l'animation quand l'élément est visible
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          startAnimation()
          observer.disconnect()
        }
      })
    },
    { threshold: 0.1 }
  )

  observer.observe(document.querySelector("span"))
})

// Redémarrer l'animation si la valeur finale change
watch(
  () => props.endValue,
  () => {
    startAnimation()
  }
)
</script>
