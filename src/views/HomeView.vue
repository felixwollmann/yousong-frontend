<script setup>
const songs = await (await fetch('http://localhost:8080/api/songs')).json()
import Song from '@/components/Song.vue'
</script>

<template>
  {{ songs }}
  <main>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-2 p-2">
      <div v-for="song in songs">
        <Song :song></Song>
      </div>
    </div>
  </main>
</template>
