<script setup>
import { ref, useTemplateRef, computed, watch, nextTick } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import SongCard from '@/components/SongCard.vue'

const route = useRoute()
const router = useRouter()

const data = ref()
const selectedGenres = ref(new Set())
const editingSelectedGenre = ref('')

// v-model="editingSelectedGenre"
// @keydown.enter="addSelectedGenre(editingSelectedGenre)"

function addSelectedGenre(genre) {
  selectedGenres.value.add(genre)
  editingSelectedGenre.value = ''
}

const searchTerm = ref('')

const pageNum = computed(() => Number(route.query.page) || 0)

const canGoBackwards = computed(() => data.value.page.number > 0)
const canGoForwards = computed(() => data.value.page.number + 1 < data.value.page.totalPages)
const lastPage = computed(() => data.value.page.totalPages - 1)

async function fetchSongs() {
  // console.log(`Fetching for pageNum: ${pageNum.value}`)
  const responseData = await (
    await fetch(
      `http://localhost:8080/api/songs?page=${pageNum.value}&q=${searchTerm.value}${[...selectedGenres.value].map((v) => `&genres=${v}`).join('')}`
    )
  ).json()
  data.value = responseData
  // console.log(data.value)
  // console.log(songs.value)
}

/** @type Ref<{id: number, title: string, artist?: string, genre?: string, length?: number}[]> */
const songs = computed(() => data.value.content)

await fetchSongs()

watch(pageNum, fetchSongs)
watch(searchTerm, fetchSongs)
watch(() => [...selectedGenres.value].join(''), fetchSongs)
watch(searchTerm, () => router.push({ query: { page: undefined } }))

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
    ...(genres.value.size > 0 ? { genres: [...genres.value] } : {}),
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
  genres.value = ''
  editingGenre.value = ''
  length.value = ''
  editingId.value = null

  await fetchSongs()
  alert('Song saved successfully')
}

function openDialogForEditing(song) {
  songName.value = song.title
  artist.value = song.artist
  genres.value = new Set(song.genres)
  length.value = song.length

  editingId.value = song.id
  dialog.value.showModal()
}

const editingId = ref(null)

const songName = ref('')
const artist = ref('')
const editingGenre = ref('')
const genres = ref(new Set())
const length = ref(0)

function addGenre(genre) {
  genres.value.add(genre)
  editingGenre.value = ''
}

import { useSound } from '@vueuse/sound'

const playingSong = ref()
const playingSongSourceUrl = computed(() =>
  playingSong.value ? `http://localhost:8080/api/songs/${playingSong.value.id}/audio` : undefined
)

const { pause, isPlaying, play, stop } = useSound(playingSongSourceUrl, {
  format: 'mp3',
  interrupt: true
})

async function playSong(song) {
  try {
    stop()
  } catch (e) {
    /* empty */
    console.error(e)
  }
  playingSong.value = song
  await nextTick()
  try {
    play()
  } catch (e) {
    /* empty */
    console.error(e)
  }
}
</script>

<template>
  <main class="p-2">
    <h1 class="text-xl">YouSong</h1>
    <input
      type="text"
      v-model="searchTerm"
      class="w-full focus:border-blue-600 p-1 transition-colors border-blue-200 border-2 rounded outline-none"
    />

    <div class="flex gap-2 flex-wrap items-center my-2 rounded">
      <div
        class="flex gap-0.5 items-center justify-center rounded-full bg-blue-200 pr-1 pl-1.5 text-sm"
        v-for="genre in selectedGenres"
        :key="genre"
      >
        <span>{{ genre }}</span>
        <button
          class="rounded-full size-3 hover:bg-blue-300 flex items-center justify-center"
          @click="selectedGenres.delete(genre)"
        >
          <span>x</span>
        </button>
      </div>
      <input
        type="text"
        class="w-32 flex-grow px-1 py-0.5 placeholder:text-gray-500 rounded border-2 border-blue-200 focus:border-blue-600 outline-none bg-transparent transition-colors"
        placeholder="Genre eg. pop, rock, jazz"
        v-model="editingSelectedGenre"
        @keydown.enter="addSelectedGenre(editingSelectedGenre)"
      />
    </div>
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-2">
      <div v-for="song in songs" :key="song.id">
        <SongCard
          :song
          @delete="deleteSong(song)"
          @edit="openDialogForEditing(song)"
          @play="playSong(song)"
        ></SongCard>
      </div>
    </div>
    <div class="" v-if="songs.length === 0">
      <template v-if="!searchTerm">
        <p>No songs :(</p>
        <p>Create one!</p>
      </template>
      <p v-else>No songs can be found. Please adjust your search.</p>
    </div>
    <button class="fixed right-10 bottom-10 bg-blue-200" @click="openDialogForNewSong">
      Add a song
    </button>
    <!-- Music player -->
    <div class="fixed bottom-0 left-0 p-10 rounded-tr-lg bg-fuchsia-700">
      <svg
        v-if="!isPlaying"
        @click="play()"
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
      >
        <path
          fill="currentColor"
          fill-opacity="0"
          stroke="currentColor"
          stroke-dasharray="40"
          stroke-dashoffset="40"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M8 6l10 6l-10 6Z"
        >
          <animate
            fill="freeze"
            attributeName="fill-opacity"
            begin="0.5s"
            dur="0.5s"
            values="0;1"
          />
          <animate fill="freeze" attributeName="stroke-dashoffset" dur="0.5s" values="40;0" />
        </path>
      </svg>
      <svg
        v-else
        @click="pause()"
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
      >
        <path
          fill="currentColor"
          d="M15 18q-.402 0-.701-.299T14 17V7q0-.402.299-.701T15 6h1.5q.402 0 .701.299T17.5 7v10q0 .402-.299.701T16.5 18zm-7.5 0q-.402 0-.701-.299T6.5 17V7q0-.402.299-.701T7.5 6H9q.402 0 .701.299T10 7v10q0 .402-.299.701T9 18z"
        />
      </svg>
    </div>
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
      <!-- <input type="text" placeholder="Genre" v-model="editingGenre" /> -->
      <div class="flex gap-2 flex-wrap items-center max-w-64 p-3 bg-slate-100 rounded">
        <div
          class="flex gap-0.5 items-center justify-center rounded-full bg-blue-200 pr-1 pl-1.5 text-sm"
          v-for="genre in genres"
          :key="genre"
        >
          <span>{{ genre }}</span>
          <button
            class="rounded-full size-3 hover:bg-blue-300 flex items-center justify-center"
            @click="genres.delete(genre)"
          >
            <span>x</span>
          </button>
        </div>
        <input
          type="text"
          class="w-32 flex-grow px-1 py-0.5 placeholder:text-gray-500 rounded border-2 border-blue-200 focus:border-blue-600 outline-none bg-transparent transition-colors"
          placeholder="Genre eg. pop, rock, jazz"
          v-model="editingGenre"
          @keydown.enter="addGenre(editingGenre)"
        />
      </div>
      <input type="number" placeholder="Length in Seconds" v-model="length" />
      <button @click="saveOrEditSong()" class="bg-blue-500 text-white rounded">Save</button>
    </div>
  </dialog>
</template>
