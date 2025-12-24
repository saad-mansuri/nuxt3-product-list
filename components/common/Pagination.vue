<template>
  <div class="flex justify-center gap-2 mt-8">
    <button
      v-for="page in totalPages"
      :key="page"
      @click="emit('change-pagination', page)"
      class="px-3 py-1 border rounded"
      :class="{
        'bg-black text-white': page === currentPage
      }"
    >
      {{ page }}
    </button>
  </div>
</template>

<script setup>

    const emit = defineEmits(['change-pagination'])
    
    defineProps({
        totalPages: Number,
        currentPage: Number
    })

</script>