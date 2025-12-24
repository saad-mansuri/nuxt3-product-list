<script setup>
    defineProps({
        product: {
            type: Object,
            required: true
        }
    })
</script>

<template>
  <div class="border rounded p-4 flex flex-col">
    <img
      :src="product.image"
      :alt="product.title"
      loading="lazy"
      class="h-40 object-contain mb-3"
    />

    <h2 class="text-sm font-semibold mb-1">
      {{ product.title }}
    </h2>

    <p class="text-xs text-gray-500 mb-2">
      {{ product.category }}
    </p>

    <p class="font-bold mt-auto">
      ${{ product.price }}
    </p>
  </div>
</template>
