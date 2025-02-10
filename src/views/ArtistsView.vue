<script setup>
import { ref } from 'vue'

const newArtistInput = ref('')

function handleCreateNewArtist() {
  const artist = newArtistInput.value
  newArtistInput.value = ''
  alert(artist)
  // tbc
}

function editArtist(id) {
  const newName = prompt('Enter new name')
  alert(newName)
}

function deleteArtist(id) {
  alert(id)
}
</script>
<template>
  <main class="my-32 w-96 mx-auto bg-slate-50 shadow p-4 rounded flex flex-col gap-2">
    <h1 class="font-bold text-xl">Artists</h1>
    <input
      type="text"
      placeholder="New Artist"
      class="ring-2 outline-none p-1 rounded focus-within:ring-4 transition-shadow m-0.5"
      v-model="newArtistInput"
      @keydown.enter.prevent="handleCreateNewArtist"
    />
    <div class="flex flex-col gap-2">
      <div
        v-for="artist of Array(5)
          .fill('')
          .map((v, idx) => `Person ${Math.random()}`)"
        :key="artist"
        class="p-2 rounded bg-gray-100 flex gap-1"
      >
        <p class="flex-1">{{ artist }}</p>
        <button @click="editArtist(1)">🖊</button>
        <button @click="deleteArtist(1)">🗑</button>
      </div>
    </div>
  </main>
</template>
