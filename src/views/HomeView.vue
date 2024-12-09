<script setup>
import { ref, useTemplateRef, computed, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import SongCard from '@/components/SongCard.vue'

const route = useRoute()
const router = useRouter()

const data = ref()

const pageNum = computed(() => Number(route.query.page) || 0)

const canGoBackwards = computed(() => !data.value.first)
const canGoForwards = computed(() => !data.value.last)
const lastPage = computed(() => data.value.totalPages - 1)

async function fetchSongs() {
  // console.log(`Fetching for pageNum: ${pageNum.value}`)
  const responseData = await (
    await fetch(`http://localhost:8080/api/songs?page=${pageNum.value}`)
  ).json()
  data.value = responseData
  // console.log(data.value)
  // console.log(songs.value)
}

/** @type Ref<{id: number, title: string, artist?: string, genre?: string, length?: number}[]> */
const songs = computed(() => data.value.content)

await fetchSongs()

watch(pageNum, fetchSongs)

async function deleteSong(song) {
  await fetch(`http://localhost:8080/api/songs/${song.id}`, { method: 'DELETE' })
  fetchSongs()
}

const dialog = useTemplateRef('dialog')

function openDialogForNewSong() {
  dialog.value.showModal()
  editingId.value = null
}

async function saveOrEditSong() {
  const newSong = {
    title: songName.value,
    ...(editingId.value ? { id: editingId.value } : {}),
    ...(artist.value ? { artist: artist.value } : {}),
    ...(genre.value ? { genre: genre.value } : {}),
    ...(length.value ? { length: length.value } : {})
  }

  dialog.value.close()

  if (newSong.id) {
    await fetch(`http://localhost:8080/api/songs/${newSong.id}`, {
      method: 'PATCH',
      body: JSON.stringify(newSong),
      headers: {
        'Content-Type': 'application/json'
      }
    })
  } else {
    await fetch('http://localhost:8080/api/songs', {
      method: 'POST',
      body: JSON.stringify(newSong),
      headers: {
        'Content-Type': 'application/json'
      }
    })
  }

  songName.value = ''
  artist.value = ''
  genre.value = ''
  length.value = ''
  editingId.value = null

  await fetchSongs()
  alert('Song saved successfully')
}

function openDialogForEditing(song) {
  songName.value = song.title
  artist.value = song.artist
  genre.value = song.genre
  length.value = song.length

  editingId.value = song.id
  dialog.value.showModal()
}

const editingId = ref(null)

const songName = ref('')
const artist = ref('')
const genre = ref('')
const length = ref(0)
</script>

<template>
  <main class="p-2">
    <h1 class="text-xl">YouSong</h1>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-2">
      <div v-for="song in songs">
        <SongCard :song @delete="deleteSong(song)" @edit="openDialogForEditing(song)"></SongCard>
      </div>
    </div>
    <div class="" v-if="songs.length === 0">
      <p>No songs :(</p>
      <p>Create one!</p>
    </div>
    <button class="fixed right-10 bottom-10 bg-blue-200" @click="openDialogForNewSong">
      Add a song
    </button>
    <div
      class="flex mt-4 gap-4 *:bg-blue-600 disabled:*:bg-gray-300 *:px-2 *:py-1 *:rounded *:text-white"
    >
      <button :disabled="!canGoBackwards" @click="router.push({ query: { page: 0 } })">
        First
      </button>
      <button :disabled="!canGoBackwards" @click="router.push({ query: { page: pageNum - 1 } })">
        Previous
      </button>
      <p class="!bg-transparent !text-black">{{ pageNum + 1 }} of {{ lastPage + 1 }}</p>
      <button :disabled="!canGoForwards" @click="router.push({ query: { page: pageNum + 1 } })">
        Next
      </button>
      <button :disabled="!canGoForwards" @click="router.push({ query: { page: lastPage } })">
        Last
      </button>
    </div>
  </main>
  <dialog class="p-4 rounded" ref="dialog">
    <div class="flex flex-col gap-2">
      <h2 class="font-bold">Create/Edit song</h2>
      <input type="text" placeholder="Song name" v-model="songName" />
      <input type="text" placeholder="Artist" v-model="artist" />
      <input type="text" placeholder="Genre" v-model="genre" />
      <input type="number" placeholder="Length in Seconds" v-model="length" />
      <button @click="saveOrEditSong()" class="bg-blue-500 text-white rounded">Save</button>
    </div>
  </dialog>
</template>
