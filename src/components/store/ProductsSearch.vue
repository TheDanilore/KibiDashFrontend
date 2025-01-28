<script setup>
import { ref, watch, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const props = defineProps({
    onClose: {
        type: Function,
        default: () => { }
    }
});

const router = useRouter();
const searchQuery = ref('');
const searchResults = ref([]);
const showResults = ref(false);
const isLoading = ref(false);
const debounceTimeout = ref(null);

const searchProducts = async () => {
    if (!searchQuery.value.trim()) {
        searchResults.value = [];
        return;
    }

    isLoading.value = true;
    try {
        const url = `/api/productos/searchByName?q=${encodeURIComponent(searchQuery.value)}`;
        const response = await axios.get(url);
        searchResults.value = response.data;
    } catch (error) {
        console.error('Error en la búsqueda:', error);
        searchResults.value = [];
    } finally {
        isLoading.value = false;
    }
};

const handleSearch = () => {
    if (searchQuery.value.trim()) {
        router.push({
            name: 'ProductsListSearch',
            query: { search: searchQuery.value }
        });
        showResults.value = false;
    }
};

const handleFocus = () => {
    showResults.value = true;
};

const goToProduct = (product) => {
    router.push(`/productos/${product.id}`);
    searchQuery.value = '';
    showResults.value = false;
    props.onClose();
};

const handleImageError = (event) => {
    event.target.src = '/placeholder-image.png'; // Imagen por defecto
};

const handleClickOutside = (event) => {
    const searchContainer = document.querySelector('.search-container');
    if (searchContainer && !searchContainer.contains(event.target)) {
        showResults.value = false;
    }
};

watch(searchQuery, (newQuery) => {
    clearTimeout(debounceTimeout.value);
    if (newQuery) {
        debounceTimeout.value = setTimeout(() => {
            showResults.value = true;
            searchProducts();
        }, 500);
    } else {
        showResults.value = false;
        searchResults.value = [];
    }
});

onMounted(() => {
    document.addEventListener('click', handleClickOutside);
});

onUnmounted(() => {
    document.removeEventListener('click', handleClickOutside);
});
</script>

<template>
    <div class="search-container">
        <div class="search-input-wrapper">
            <input type="text" v-model="searchQuery" @focus="handleFocus" @keyup.enter="handleSearch"
                placeholder="Buscar productos..." class="search-input" />
            <button @click="handleSearch" class="search-button">
                🔍
            </button>
            <span v-if="isLoading" class="loading-indicator">🔄</span>
        </div>

        <div class="search-results-wrapper" v-show="showResults">
            <!-- Resultados de búsqueda -->
            <div v-if="searchResults.length > 0" class="dropdown-content search-results">
                <div v-for="producto in searchResults" :key="producto.id" @click="goToProduct(producto)"
                    class="search-result-item">
                    <img v-if="producto.imagen" :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${producto.imagen}`"
                        alt="Imagen del producto" class="search-result-image" @error="handleImageError" />
                    <!--  <img v-if="producto.imagen" :src="`http://localhost:8000/${producto.imagen}`"
                        alt="Imagen del producto" class="search-result-image" @error="handleImageError" /> -->
                    <div class="search-result-info">
                        <span class="product-name">{{ producto.nombre }}</span>
                    </div>
                </div>
            </div>

            <!-- Mensaje sin resultados -->
            <div v-if="searchQuery && searchResults.length === 0 && !isLoading" class="dropdown-content no-results">
                No se encontraron productos
            </div>
        </div>
    </div>
</template>


<style scoped>
.search-container {
    position: relative;
    width: 100%;
    max-width: 600px;
}

.search-input-wrapper {
    position: relative;
    display: flex;
    align-items: center;
}

.search-input {
    width: 100%;
    padding: 8px 40px 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 14px;
}

.search-button {
    position: absolute;
    right: 8px;
    background: none;
    border: none;
    cursor: pointer;
}

.loading-indicator {
    position: absolute;
    right: 40px;
}

.search-results-wrapper {
    position: absolute;
    top: 100%;
    left: 0;
    width: 100%;
    background: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-top: 4px;
    max-height: 400px;
    overflow-y: auto;
    z-index: 1000;
}

.search-result-item {
    display: flex;
    align-items: center;
    padding: 8px;
    cursor: pointer;
    border-bottom: 1px solid #eee;
}

.search-result-item:hover {
    background-color: #f5f5f5;
}

.search-result-image {
    width: 50px;
    height: 50px;
    object-fit: cover;
    margin-right: 12px;
    border-radius: 4px;
}

.search-result-info {
    flex: 1;
}

.product-name {
    font-size: 14px;
    color: #333;
}

.no-results {
    padding: 12px;
    text-align: center;
    color: #666;
}
</style>