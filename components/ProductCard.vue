<script setup>
    defineProps({
        product: {
            type: Object,
            required: true
        }
    })
</script>

<template>
  <div
    class="group bg-white rounded-2xl shadow-md hover:shadow-xl transition-all duration-300 overflow-hidden"
  >
    <!-- Image -->
    <div class="h-52 bg-gray-100 flex items-center justify-center overflow-hidden">
      <img
        :src="product.image"
        :alt="product.title"
        loading="lazy"
        class="h-40 object-contain transition-transform duration-300 group-hover:scale-110"
      />
    </div>

    <!-- Content -->
    <div class="p-4 flex flex-col h-[150px]">
      <h2
        class="text-sm font-semibold text-gray-800 mb-1 line-clamp-2"
        title="product.title"
      >
        {{ product.title }}
      </h2>

      <p class="text-xs uppercase tracking-wide text-gray-400 mb-3">
        {{ product.category }}
      </p>

      <div class="mt-auto flex items-center justify-between">
        <p class="text-lg font-bold text-black">
          ${{ product.price }}
        </p>

        <span
          class="text-xs px-3 py-1 rounded-full bg-black text-white opacity-0 group-hover:opacity-100 transition-opacity cursor-pointer"
        >
          View
        </span>
      </div>
    </div>
  </div>
</template>
