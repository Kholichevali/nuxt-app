<template>
  <div class="card">
    <form @submit.prevent="applySearch" class="search-form">
      <input v-model="searchInput" type="text" placeholder="Search..." class="search-input"/>
      <button type="submit" class="button button-default">Search</button>
      <button v-if="searchQuery" type="button" @click="clearSearch" class="button button-secondary">Clear</button>
    </form>

    <table class="product-table">
      <thead>
      <tr>
        <th v-for="header in headers" :key="header.key" :class="{ 'description-header': header.key === 'description' }">
          <button @click="sortBy(header.key)" class="sort-button">
            {{ header.title }}
            <svg v-if="sortKey === header.key" xmlns="http://www.w3.org/2000/svg" class="sort-icon" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" :d="sortAsc ? 'M5 15l7-7 7 7' : 'M19 9l-7 7-7-7'"/>
            </svg>
          </button>
        </th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="product in paginatedProducts" :key="product.id" class="product-row">
        <td class="product-cell">
          <p class="product-title">
            {{ product.title }}
          </p>
        </td>
        <td class="product-cell description-cell">
          <p class="product-description">
            {{ product.description }}
          </p>
        </td>
        <td class="product-cell">
          <p class="product-price">
            ${{ product.price }}
          </p>
        </td>
        <td class="product-cell">
          <p class="product-rating" :class="{ 'low-rating': product.rating < 4.5, 'high-rating': product.rating >= 4.5 }">
            {{ product.rating }}
          </p>
        </td>
        <td class="product-cell">
          <p class="product-brand">
            {{ product.brand }}
          </p>
        </td>
        <td class="product-cell">
          <p class="product-category">
            {{ product.category }}
          </p>
        </td>
        <td class="product-cell">
          <img :src="product.thumbnail" :alt="product.title" class="product-thumbnail" loading="lazy">
        </td>
      </tr>
      </tbody>
    </table>

    <div class="pagination">
      <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1" class="button button-success">Previous</button>
      <span class="pagination-info">Page {{ currentPage }} of {{ totalPages }}</span>
      <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages" class="button button-success">
        Next
      </button>
    </div>
  </div>
</template>
<script setup>
import { ref, computed } from 'vue';

const { data: apiData } = await useFetch('https://dummyjson.com/products');
const products = ref(apiData?.value?.products || []);

const searchInput = ref('');
const searchQuery = ref('');
const currentPage = ref(1);
const perPage = 10;
const sortKey = ref('');
const sortAsc = ref(true);

const headers = [
  { key: 'title', title: 'Title' },
  { key: 'description', title: 'Description' },
  { key: 'price', title: 'Price' },
  { key: 'rating', title: 'Rating' },
  { key: 'brand', title: 'Brand' },
  { key: 'category', title: 'Category' },
  { key: 'thumbnail', title: '' },
];

function applySearch() {
  searchQuery.value = searchInput.value.trim();
  currentPage.value = 1;
}

function clearSearch() {
  searchInput.value = '';
  searchQuery.value = '';
  currentPage.value = 1;
}

const filteredProducts = computed(() => {
  if (!searchQuery.value) return products.value;
  return products.value.filter(p =>
      Object.values(p).some(val =>
          String(val).toLowerCase().includes(searchQuery.value.toLowerCase())
      )
  );
});

function sortBy(key) {
  if (sortKey.value === key) {
    sortAsc.value = !sortAsc.value;
  } else {
    sortKey.value = key;
    sortAsc.value = true;
  }
}

const sortedProducts = computed(() => {
  if (!sortKey.value) return filteredProducts.value;

  return [...filteredProducts.value].sort((a, b) => {
    const valA = a[sortKey.value];
    const valB = b[sortKey.value];

    if (typeof valA === 'string') {
      return sortAsc.value
          ? valA.localeCompare(valB)
          : valB.localeCompare(valA);
    } else {
      return sortAsc.value ? valA - valB : valB - valA;
    }
  });
});

const totalPages = computed(() =>
    Math.ceil(sortedProducts.value.length / perPage)
);

const paginatedProducts = computed(() =>
    sortedProducts.value.slice(
        (currentPage.value - 1) * perPage,
        currentPage.value * perPage
    )
);

function changePage(page) {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
  }
}
</script>