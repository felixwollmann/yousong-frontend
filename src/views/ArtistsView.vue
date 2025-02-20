<script setup>
import { ref } from 'vue'

const newArtistInput = ref('')

const artists = ref(await (await fetch('http://localhost:8080/api/artists')).json())

async function handleCreateNewArtist() {
  const name = newArtistInput.value
  newArtistInput.value = ''
  artists.value.push(
    await (
      await fetch('http://localhost:8080/api/artists', {
        method: 'POST',
        body: JSON.stringify({ name }),
        headers: {
          'Content-Type': 'application/json'
        }
      })
    ).json()
  )
}

async function editArtist(id) {
  const name = prompt('Enter new name', artists.value.find((v) => v.id === id).name)
  await fetch('http://localhost:8080/api/artists/' + id, {
    method: 'PATCH',
    body: JSON.stringify({ id, name }),
    headers: {
      'Content-Type': 'application/json'
    }
  })
  artists.value = await (await fetch('http://localhost:8080/api/artists')).json()
}

async function deleteArtist(id) {
  await fetch('http://localhost:8080/api/artists/' + id, {
    method: 'DELETE'
  })
  artists.value = artists.value.filter((artist) => artist.id !== id)
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
      <div v-for="artist of artists" :key="artist.id" class="p-2 rounded bg-gray-100 flex gap-1">
        <p class="flex-1">{{ artist.name }}</p>
        <button @click="editArtist(artist.id)">🖊</button>
        <button @click="deleteArtist(artist.id)">🗑</button>
      </div>
    </div>
  </main>
</template>
