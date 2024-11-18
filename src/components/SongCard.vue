<script setup>
const props = defineProps({
  song: Object
})

const emit = defineEmits(['delete', 'edit'])
</script>

<template>
  <div class="bg-slate-100 p-4 rounded relative">
    <h2 class="font-bold">{{ song.title }}</h2>
    <p>{{ song.artist }}</p>
    <p>{{ song.genre }}</p>
    <p>{{ song.length }} Sekunden</p>
    <button
      @click="emit('delete')"
      class="absolute bottom-0 right-0 bg-slate-300 px-1 py-0.5 rounded"
    >
      Delete
    </button>
    <button @click="emit('edit')" class="absolute top-0 right-0 bg-slate-300 px-1 py-0.5 rounded">
      Edit
    </button>
  </div>
</template>
