<template>
  <header class="center-align">
    <h1 class="small">YouTube Share Time Adder</h1>
  </header>
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
    <p>
      <button @click="setClipboard(sharedUrl)" target="_blank" rel="noopener noreferrer">
        Timed URL: <span class="bold">{{ sharedUrl }}</span> (click to copy)
      </button>
    </p>
  </main>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'
import { useRoute } from 'vue-router'

const videoUrl = ref('')
const timeValue = ref('00:00:00')
const route = useRoute()

watchEffect(() => {
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
    if (videoUrl.value !== detectedUrl) {
      videoUrl.value = detectedUrl
    }
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
    return 'https://youtu.be/dQw4w9WgXcQ'
  }
  try {
    const url = new URL(videoUrl.value)
    url.searchParams.set('t', totalSeconds.value)
    url.searchParams.delete('si')
    return url.toString()
  } catch (error) {
    return 'Invalid YouTube URL'
  }
})

async function setClipboard(text) {
  const type = 'text/plain'
  const clipboardItemData = {
    [type]: text,
  }
  const clipboardItem = new ClipboardItem(clipboardItemData)
  await navigator.clipboard.write([clipboardItem])
}
</script>
