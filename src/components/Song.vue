<script setup>
const props = defineProps({
  song: Object
})
</script>

<template>
  <div class="bg-slate-200 p-4 rounded">
    <h2 class="font-bold">{{ song.title }}</h2>
    <p>{{ song.artist }}</p>
    <p>{{ song.genre }}</p>
    <p>{{ song.length }} Sekunden</p>
  </div>
</template>
