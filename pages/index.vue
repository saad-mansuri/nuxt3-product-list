<template>
  <div class="max-w-7xl mx-auto p-6">
    <h1 class="text-2xl font-bold mb-6">Product List</h1>

    <!-- Search & Filter -->
    <div class="flex flex-col md:flex-row gap-4 mb-6">
      <input
        v-model="searchProductsQuery"
        type="text"
        placeholder="Search by title..."
        aria-label="Search products"
        class="border rounded px-3 py-2 w-full md:w-1/2"
      />

      <select
        v-model="selectedCategory"
        aria-label="Filter by category"
        class="border rounded px-3 py-2 w-full md:w-1/3"
      >
        <option v-for="cat in filterCategories" :key="cat" :value="cat">
          {{ cat }}
        </option>
      </select>
    </div>

    <!-- States -->
    <p v-if="fetchProductsLoading">Loading products...</p>
    <p v-if="error" class="text-red-500">{{ error }}</p>

    <!-- Products -->
    <div
      v-if="!fetchProductsLoading"
      class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6"
    >
      <ProductCard
        v-for="product in paginatedProducts"
        :key="product.id"
        :product="product"
      />
    </div>

    <!-- Pagination -->
    <Pagination
      v-if="totalPages > 1"
      :totalPages="totalPages"
      :currentPage="currentPage"
      @change-pagination="(page) => (currentPage = page)"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import ProductCard from "~/components/ProductCard.vue";
import Pagination from "~/components/common/Pagination.vue";

const products = ref([]);
const fetchProductsLoading = ref(false);
const error = ref(null);

const searchProductsQuery = ref("");
const selectedCategory = ref("all");
const currentPage = ref(1);

const ITEMS_PER_PAGE = 10;

onMounted(async () => {
  fetchProducts();
});

const fetchProducts = async () => {
  fetchProductsLoading.value = true;
  try {
    const res = await fetch("https://fakestoreapi.com/products");
    products.value = await res.json();
  } catch (err) {
    error.value = "Failed to load products";
  } finally {
    fetchProductsLoading.value = false;
  }
};

/* Categories */
const filterCategories = computed(() => {
  const unique = new Set(products.value.map((p) => p.category));
  return ["all", ...unique];
});

/* Search + Filter */
const filteredProducts = computed(() => {
  let productsList = products.value;

  if (selectedCategory.value !== "all") {
    productsList = productsList.filter(
      (p) => p.category === selectedCategory.value
    );
  }

  if (searchProductsQuery.value.trim()) {
    currentPage.value = 1;
    productsList = productsList.filter((p) =>
      p.title.toLowerCase().includes(searchProductsQuery.value.toLowerCase())
    );
  }

  return productsList;
});

/* Pagination */
const totalPages = computed(() =>
  Math.ceil(filteredProducts.value.length / ITEMS_PER_PAGE)
);

const paginatedProducts = computed(() => {
  const startIndex = (currentPage.value - 1) * ITEMS_PER_PAGE;
  return filteredProducts.value.slice(startIndex, startIndex + ITEMS_PER_PAGE);
});
</script>
