<template>
  <div class="min-h-screen bg-gray-50">
    <div class="max-w-7xl mx-auto p-6">

      <!-- Search & Filter -->
      <div class="flex flex-wrap items-center gap-3 bg-white p-3 rounded-xl shadow-sm justify-end mb-6">

        <!-- Search -->
        <div class="relative w-full sm:w-64">
          <input
            v-model="searchProducts"
            @input="searchProduct"
            type="text"
            placeholder="Search products..."
            class="border rounded-lg pl-3 pr-9 py-2 w-full text-sm focus:outline-none focus:ring-2 focus:ring-black"
          />

          <button
            v-if="searchProducts"
            @click="resetProductSearch"
            class="absolute right-2 top-1/2 -translate-y-1/2 text-gray-400 hover:text-black"
          >
            ✕
          </button>
        </div>

        <!-- Category -->
         <div
          class="relative w-full sm:w-48"
          :class="{ 'cat-loading': isCategoryLoading }"
        >
          <select
            v-model="activeCategory"
            @change="changeCategory"
            class="border rounded-lg px-3 py-2 text-sm w-full sm:w-48 focus:outline-none focus:ring-2 focus:ring-black"
          >
            <option value="all">All Categories</option>
            <option
              v-for="cat in categoryList"
              :key="cat"
              :value="cat.slug"
            >
              {{ cat.name }}
            </option>
          </select>
        </div>
      </div>

      <!-- Error -->
      <div v-if="error" class="mt-16 flex justify-center">
        <div class="max-w-md w-full text-center bg-red-50 border border-red-200 rounded-lg p-6">
          <p class="text-red-600 font-medium mb-3">
            {{ error }}
          </p>
          <p class="text-xs text-slate-500">
            Please check your internet connection or try again later.
          </p>
        </div>
      </div>

      <!-- Products -->
      <div
        class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-8 product-grid"
        :class="{ loading: isProductsLoading }"
      >
        <template v-if="filteredProducts.length">
          <ProductCard
            v-for="product in filteredProducts"
            :key="product.id"
            :product="product"
          />
        </template>

        <p v-else class="col-span-full text-center text-gray-500 py-12">
          No products found
        </p>
      </div>

      <!-- Pagination -->
      <div
        v-if="totalProducts > pageSize"
        class="flex justify-center gap-3 mt-8"
      >
        <button
          :disabled="offset === 0"
          @click="goToPreviousPage"
          class="px-4 py-2 border rounded disabled:opacity-40"
        >
          Prev
        </button>

        <span class="text-sm flex items-center">
          Page {{ currentPageNumber }}
        </span>

        <button
          :disabled="offset + pageSize >= totalProducts"
          @click="goToNextPage"
          class="px-4 py-2 border rounded disabled:opacity-40"
        >
          Next
        </button>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import ProductCard from "~/components/ProductCard.vue";

/* =====================
   STATE
===================== */
const productList = ref([]);
const categoryList = ref([]);
const error = ref(null);
const isProductsLoading = ref(false);
const isCategoryLoading = ref(false);

const searchProducts = ref("");
const activeCategory = ref("all");

const pageSize = 12;
const offset = ref(0);
const totalProducts = ref(0);
const currentPageNumber = ref(1);

/* =====================
   COMPUTED
===================== */
const filteredProducts = computed(() => {
  return productList.value ?? [];
});

/* =====================
   API HELPER
===================== */
const fetchProductList = async (url) => {
  try {
    error.value = null;
    isProductsLoading.value = true;

    const res = await fetch(url);
    const data = await res.json();

    productList.value = data?.products ?? [];
    totalProducts.value = data?.total ?? 0;
  } catch {
    error.value = "Failed to load products";
    productList.value = [];
    totalProducts.value = 0;
  } finally {
    isProductsLoading.value = false;
  }
};

/* =====================
   INITIAL LOAD
===================== */
onMounted(async () => {
  await getCategories();
  await getAllProductList();
});

/* =====================
   LOADERS
===================== */
const getAllProductList = async () => {
  offset.value = 0;
  currentPageNumber.value = 1;

  await fetchProductList(
    `https://dummyjson.com/products?limit=${pageSize}&skip=${offset.value}`
  );
};

const getCategories = async () => {
  isCategoryLoading.value = true;
  const res = await fetch("https://dummyjson.com/products/categories");
  isCategoryLoading.value = false;
  categoryList.value = await res.json();
};

const searchProduct = async () => {
  offset.value = 0;
  currentPageNumber.value = 1;

  if (!searchProducts.value.trim()) {
    await getAllProductList();
    return;
  }

  activeCategory.value = "all";

  await fetchProductList(
    `https://dummyjson.com/products/search?q=${searchProducts.value}&limit=${pageSize}&skip=0`
  );
};

/* =====================
   CATEGORY
===================== */
const changeCategory = async () => {
  searchProducts.value = "";
  offset.value = 0;
  currentPageNumber.value = 1;

  if (activeCategory.value === "all") {
    await getAllProductList();
    return;
  }

  await fetchProductList(
    `https://dummyjson.com/products/category/${activeCategory.value}?limit=${pageSize}&skip=0`
  );
};

/* =====================
   PAGINATION
===================== */
const goToNextPage = async () => {
  offset.value += pageSize;
  currentPageNumber.value++;
  await fetchProductsOnPageChange();
};

const goToPreviousPage = async () => {
  offset.value -= pageSize;
  currentPageNumber.value--;
  await fetchProductsOnPageChange();
};

const resetProductSearch = async () => {
  searchProducts.value = "";
  await getAllProductList();
};


const fetchProductsOnPageChange = async () => {
  if (searchProducts.value.trim()) {
    await fetchProductList(
      `https://dummyjson.com/products/search?q=${searchProducts.value}&limit=${pageSize}&skip=${offset.value}`
    );
  } else if (activeCategory.value !== "all") {
    await fetchProductList(
      `https://dummyjson.com/products/category/${activeCategory.value}?limit=${pageSize}&skip=${offset.value}`
    );
  } else {
    await fetchProductList(
      `https://dummyjson.com/products?limit=${pageSize}&skip=${offset.value}`
    );
  }
};
</script>
