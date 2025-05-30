<template>
  <main class="responsive center-align top-margin bottom-margin">
    <div class="field label border">
      <input type="text" id="videoUrlInput" v-model="videoUrl" />
      <label for="videoUrlInput">YouTube Video URL:</label>
    </div>

    <div class="field label prefix border">
      <i>schedule</i>
      <input type="time" id="timeInput" v-model="timeValue" step="1" />

      <label for="timeInput">Time:</label>
    </div>
    <p>Shared URL: {{ sharedUrl }}</p>
  </main>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const videoUrl = ref('')
const timeValue = ref('00:00:00')

onMounted(() => {
  const route = useRoute()
  const queryUrl = route.query.url
  const queryText = route.query.text

  let detectedUrl = queryUrl

  if (
    !detectedUrl &&
    queryText &&
    typeof queryText === 'string' &&
    (queryText.startsWith('http://') || queryText.startsWith('https://'))
  ) {
    detectedUrl = queryText
  }

  if (detectedUrl) {
    videoUrl.value = detectedUrl
  } else {
    videoUrl.value = 'https://youtu.be/dQw4w9WgXcQ?si=PzaFnbcKc8GcWiqH'
  }
})

const totalSeconds = computed(() => {
  if (!timeValue.value) {
    return 0
  }
  const parts = timeValue.value.split(':')
  const hours = parseInt(parts[0], 10) || 0
  const minutes = parseInt(parts[1], 10) || 0
  const seconds = parseInt(parts[2], 10) || 0
  return hours * 3600 + minutes * 60 + seconds
})

const sharedUrl = computed(() => {
  if (!videoUrl.value) {
    return ''
  }
  try {
    const url = new URL(videoUrl.value)
    url.searchParams.set('t', totalSeconds.value)
    return url.toString()
  } catch (error) {
    console.error('Invalid URL:', error)
    return 'Invalid YouTube URL'
  }
})
</script>
