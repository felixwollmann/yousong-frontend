<script setup>
import { useDropZone } from '@vueuse/core'
import { useTemplateRef } from 'vue'

const props = defineProps({
  song: Object
})

const emit = defineEmits(['delete', 'edit', 'play'])

const dropZoneElement = useTemplateRef('dropZoneElement')

const { isOverDropZone } = useDropZone(dropZoneElement, {
  /** @param {File[] | null} files */
  async onDrop(files) {
    const formData = new FormData()
    formData.append('file', files[0])

    await fetch(`http://localhost:8080/api/songs/${props.song.id}/audio`, {
      method: 'POST',
      body: formData
    })
  },
  // specify the types of data to be received.
  dataTypes: ['audio/mpeg', 'audio/mp3'],
  // control multi-file drop
  multiple: false,
  // whether to prevent default behavior for unhandled events
  preventDefaultForUnhandled: false
})
</script>

<template>
  <div
    class="bg-slate-100 p-4 rounded relative transition-colors"
    :class="{ '!bg-blue-300': isOverDropZone }"
    ref="dropZoneElement"
  >
    <h2 class="font-bold">{{ song.title }}</h2>
    <p>{{ song.artist }}</p>
    <p>{{ song.genres.join(', ') }}</p>
    <p>{{ song.length }} Sekunden</p>

    <div class="flex absolute bottom-0 right-0 gap-1">
      <button @click="emit('delete')" class="bg-slate-300 px-1 py-0.5 rounded">Delete</button>
      <button @click="emit('edit')" class="bg-slate-300 px-1 py-0.5 rounded">Edit</button>
      <button @click="emit('play')" class="bg-slate-300 px-1 py-0.5 rounded">Play</button>
    </div>
  </div>
</template>
