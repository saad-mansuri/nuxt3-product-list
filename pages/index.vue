<template>
  <div class="min-h-screen bg-gray-50">
    <div class="max-w-7xl mx-auto p-6">
      <!-- <h1 class="text-3xl font-extrabold mb-8 text-gray-800">
        🛍️ Product List
      </h1> -->

      <div class="flex flex-col gap-2 mb-10">
        <div class="flex items-center gap-3">
          <div
            class="h-10 w-10 flex items-center justify-center rounded-lg bg-indigo-50"
          >
            🛍️
          </div>

          <h1 class="text-3xl font-semibold text-slate-800">
            Product Explorer
          </h1>
        </div>

        <p class="text-slate-500 text-sm max-w-xl">
          Browse high-quality products, filter by category, and find what fits
          your needs — fast and effortlessly.
        </p>

        <!-- subtle divider -->
        <div class="h-px bg-slate-200 mt-4 w-24"></div>
      </div>

      <!-- Search & Filter -->
      <div class="flex justify-end mb-8">
        <div
          class="flex flex-wrap items-center gap-3 bg-white p-3 rounded-xl shadow-sm w-full lg:w-auto"
        >
          <!-- Search Input -->
          <div class="relative w-full sm:w-64">
            <input
              v-model="searchProductsQuery"
              type="text"
              :disabled="!!error"
              placeholder="Search products..."
              aria-label="Search products"
              class="border rounded-lg pl-3 pr-9 py-2 w-full text-sm focus:outline-none focus:ring-2 focus:ring-black"
            />

            <!-- Clear (X) Icon -->
            <button
              v-if="searchProductsQuery"
              @click="resetSearch"
              type="button"
              class="absolute right-2 top-1/2 -translate-y-1/2 text-gray-400 hover:text-black transition"
              aria-label="Clear search"
            >
              ✕
            </button>
          </div>

          <!-- Category Filter -->
          <select
            v-model="selectedCategory"
            :disabled="!!error"
            aria-label="Filter by category"
            class="border rounded-lg px-3 py-2 text-sm w-full sm:w-48 focus:outline-none focus:ring-2 focus:ring-black"
          >
            <option v-for="cat in filterCategories" :key="cat" :value="cat">
              {{ cat }}
            </option>
          </select>

          <!-- Price Filter -->
          <select
            v-model="priceFilter"
            class="border rounded-lg px-3 py-2 text-sm w-full sm:w-40 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <option value="all">All Prices</option>
            <option value="0-50">Under $50</option>
            <option value="50-100">$50 – $100</option>
            <option value="100-500">$100 – $500</option>
            <option value="500+">$500+</option>
          </select>

          <!-- Sort -->
          <select
            v-model="sortBy"
            class="border rounded-lg px-3 py-2 text-sm w-full sm:w-36 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <option value="default">Sort</option>
            <option value="price-asc">Price ↑</option>
            <option value="price-desc">Price ↓</option>
            <option value="name-asc">Name A–Z</option>
          </select>
        </div>
      </div>

      <!-- States -->
      <div v-if="error" class="mt-16 flex justify-center">
        <div
          class="max-w-md w-full text-center bg-red-50 border border-red-200 rounded-lg p-6"
        >
          <!-- API Error Message -->
          <p class="text-red-600 font-medium mb-3">
            {{ error }}
          </p>

          <!-- Helper text -->
          <p class="text-xs text-slate-500">
            Please check your internet connection or try again later.
          </p>
        </div>
      </div>

      <!-- <p v-if="error" class="text-red-500 font-medium">
        {{ error }}
      </p> -->

      <!-- Products -->
      <div
        class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8 product-grid"
        :class="{ loading: fetchProductsLoading }"
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
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
const config = useRuntimeConfig();
import ProductCard from "~/components/ProductCard.vue";
// const LazyProductCard = defineAsyncComponent(() =>
//   import('~/components/ProductCard.vue')
// )
import Pagination from "~/components/common/Pagination.vue";

const products = ref([]);
const fetchProductsLoading = ref(false);
const error = ref(null);

const searchProductsQuery = ref("");
const selectedCategory = ref("all");
const currentPage = ref(1);
const priceFilter = ref("all");
const sortBy = ref("default");

const ITEMS_PER_PAGE = 10;

onMounted(async () => {
  console.log("config.public.apiUrl :", config.public.apiUrl);

  fetchProducts();
});

const fetchProducts = async () => {
  fetchProductsLoading.value = true;
  try {
    const res = await fetch(`${config.public.apiUrl}/products`);
    // if (!res.ok) {
    //   const errorText = await res.text();
    //   throw new Error(errorText || `Request failed with status ${res.status}`);
    // }
    products.value = await res.json();
  } catch (err) {
    // error.value = err?.message || "Failed to load products";
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

  // Price Filter
  if (priceFilter.value !== "all") {
    productsList = productsList.filter((p) => {
      const price = p.price;
      if (priceFilter.value === "0-50") return price < 50;
      if (priceFilter.value === "50-100") return price >= 50 && price <= 100;
      if (priceFilter.value === "100-500") return price > 100 && price <= 500;
      if (priceFilter.value === "500+") return price > 500;
      return true;
    });
  }

  // Sorting
  if (sortBy.value === "price-asc") {
    productsList.sort((a, b) => a.price - b.price);
  }
  if (sortBy.value === "price-desc") {
    productsList.sort((a, b) => b.price - a.price);
  }
  if (sortBy.value === "name-asc") {
    productsList.sort((a, b) => a.title.localeCompare(b.title));
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

const resetSearch = () => {
  searchProductsQuery.value = "";
  currentPage.value = 1;
};
</script>
