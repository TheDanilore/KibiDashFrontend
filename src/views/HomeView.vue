<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';

const router = useRouter();
const loading = ref(false);
const error = ref(null);
const featuredProducts = ref([]);
const categories = ref([]);

const fetchCategories = async () => {
  try {
    const response = await axios.get('/api/categoriaproductos');
    categories.value = response.data;
  } catch (error) {
    console.error('Error al cargar categorías:', error);
  }
};


const promotions = ref([
  {
    id: 1,
    title: '¡Ofertas de Temporada!',
    description: 'Hasta 40% de descuento en productos seleccionados',
    buttonText: 'Ver Ofertas',
    class: 'bg-primary',
    link: '/ofertas'
  },
  {
    id: 2,
    title: 'Nuevas Llegadas',
    description: 'Descubre las últimas tendencias',
    buttonText: 'Explorar',
    class: 'bg-success',
    link: '/nuevas-llegadas'
  }
]);

const activeIndex = ref({});

const obtenerProductosDestacados = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await axios.get(`/api/productos?page=${1}`, {
      params: {
        estado: 'Activo'
      }  // Agregar el parámetro de categoría
    });
    const productos = response.data.data;

    for (const producto of productos) {
      const inventarioResponse = await axios.get(`/api/inventario/producto/${producto.id}`);
      producto.inventario = inventarioResponse.data;
      activeIndex.value[producto.id] = 0;
    }

    featuredProducts.value = productos;
  } catch (err) {
    console.error('Error al obtener productos destacados:', err);
    error.value = 'No se pudieron cargar los productos destacados. Por favor, intente más tarde.';
  } finally {
    loading.value = false;
  }
};

const nextImage = (productoId) => {
  const producto = featuredProducts.value.find(p => p.id === productoId);
  if (producto && producto.imagenes?.length > 0) {
    activeIndex.value[productoId] = (activeIndex.value[productoId] + 1) % producto.imagenes.length;
  }
};

const prevImage = (productoId) => {
  const producto = featuredProducts.value.find(p => p.id === productoId);
  if (producto && producto.imagenes?.length > 0) {
    activeIndex.value[productoId] =
      (activeIndex.value[productoId] - 1 + producto.imagenes.length) % producto.imagenes.length;
  }
};

const verDetalle = (id) => {
  router.push(`/productos/${id}`);
};

const navegarACategoria = (categoria) => {
  if (categoria && categoria.descripcion) {
    router.push(`/productos?categoria=${categoria.id}`);
  }
};

const navegarABanner = (link) => {
  router.push(link);
};

onMounted(() => {
  fetchCategories();
  obtenerProductosDestacados();
});
</script>

<template>
  <main>
    <!-- Hero Section -->
    <section class="hero-section">
      <div class="container py-5">
        <div class="text-center mb-5">
          <h1 class="display-4 fw-bold">Bienvenido a Nuestra Tienda Online</h1>
          <p class="lead">Descubre productos de calidad a precios competitivos</p>
        </div>

        <!-- Promotional Banners -->
        <div class="row g-4">
          <div v-for="promo in promotions" :key="promo.id" class="col-md-6">
            <div :class="['promo-card', promo.class]">
              <h3 class="promo-title">{{ promo.title }}</h3>
              <p class="promo-description">{{ promo.description }}</p>
              <button @click="navegarABanner(promo.link)" class="btn btn-light rounded-pill">
                {{ promo.buttonText }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Featured Products Section -->
    <section class="featured-section container py-5">
      <h2 class="section-title mb-4">Productos Destacados</h2>

      <div v-if="loading" class="text-center py-5">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Cargando...</span>
        </div>
      </div>

      <div v-else-if="error" class="alert alert-danger" role="alert">
        {{ error }}
      </div>

      <div v-else class="row g-4">
        <div v-for="producto in featuredProducts" :key="producto.id" class="col-md-6 col-lg-4">
          <div class="product-card">
            <div class="product-image-container">

              <img v-if="producto.imagenes?.length"
                :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${producto.imagenes[activeIndex[producto.id]]?.url}`"
                :alt="producto.nombre" class="product-image" @click="verDetalle(producto.id)">
              <!-- <img v-if="producto.imagenes?.length"
                :src="`http://localhost:8000/${producto.imagenes[activeIndex[producto.id]]?.url}`"
                :alt="producto.nombre" class="product-image" @click="verDetalle(producto.id)"> -->

              <button @click.stop="prevImage(producto.id)" class="carousel-control prev">❮</button>
              <button @click.stop="nextImage(producto.id)" class="carousel-control next">❯</button>

              <div class="image-counter">
                {{ activeIndex[producto.id] + 1 }}/{{ producto.imagenes?.length }}
              </div>
            </div>

            <div class="product-info">
              <h3 class="product-title">{{ producto.nombre }}</h3>
              <p class="product-description">{{ producto.descripcion }}</p>
              <div class="d-flex justify-content-between align-items-center">
                <span class="product-price">
                  S/{{ producto.inventario?.[0]?.precio_unitario || producto.precio_unitario_maximo }}
                </span>
                <button @click="verDetalle(producto.id)" class="btn btn-primary">
                  Ver Detalle
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Categories Section -->
    <section class="categories-section py-5">
      <div class="container">
        <h2 class="section-title mb-4">Categorías</h2>
        <div class="row g-4">
          <div v-for="category in categories" :key="category.id" class="col-6 col-md-3">
            <div class="category-card" @click="category.descripcion ? navegarACategoria(category) : null">
              <span class="category-icon">{{ category.icon }}</span>
              <h3 class="category-title">{{ category.descripcion }}</h3>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.hero-section {
  background-color: #f8f9fa;
  padding: 2rem 0;
}

.section-title {
  font-size: 2rem;
  font-weight: bold;
  color: #333;
}

/* Promotional Cards */
.promo-card {
  padding: 2rem;
  border-radius: 1rem;
  color: white;
  transition: transform 0.3s ease;
}

.promo-card:hover {
  transform: translateY(-5px);
}

.promo-title {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.promo-description {
  margin-bottom: 1.5rem;
}

/* Product Cards */
.product-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: box-shadow 0.3s ease;
  height: 100%;
}

.product-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.product-image-container {
  position: relative;
  height: 300px;
  overflow: hidden;
}

.product-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  cursor: pointer;
}

.carousel-control {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  cursor: pointer;
  transition: background-color 0.3s;
}

.carousel-control:hover {
  background: rgba(0, 0, 0, 0.7);
}

.carousel-control.prev {
  left: 10px;
}

.carousel-control.next {
  right: 10px;
}

.image-counter {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 0.875rem;
}

.product-info {
  padding: 1.5rem;
}

.product-title {
  font-size: 1.25rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: #333;
}

.product-description {
  color: #666;
  margin-bottom: 1rem;
}

.product-price {
  font-size: 1.5rem;
  font-weight: bold;
  color: #333;
}

/* Category Cards */
.category-card {
  background: white;
  border-radius: 8px;
  padding: 1.5rem;
  text-align: center;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.category-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.category-icon {
  font-size: 2.5rem;
  display: block;
  margin-bottom: 1rem;
}

.category-title {
  font-size: 1.25rem;
  margin: 0;
  color: #333;
}
</style>