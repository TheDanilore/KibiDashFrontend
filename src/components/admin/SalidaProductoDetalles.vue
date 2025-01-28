<script>
import AdminMenu from './AdminMenu.vue';
export default {
    components: {
        AdminMenu,
    },
    props: {
        id: {
            type: String,
            required: false
        }
    },
    data() {
        return {
            salida: null
        };
    },
    mounted() {
        try {
            this.salida = JSON.parse(this.$route.query.salida);
        } catch (error) {
            console.error("Error parsing salida:", error);
            this.salida = null;
        }
    },
    methods: {
        goBack() {
            this.$router.push({ name: 'SalidaProductoList' });
        },
        getTextColor(hexColor) {
            const r = parseInt(hexColor.substr(1, 2), 16);
            const g = parseInt(hexColor.substr(3, 2), 16);
            const b = parseInt(hexColor.substr(5, 2), 16);
            const brightness = (r * 299 + g * 587 + b * 114) / 1000;
            return brightness > 128 ? '#000000' : '#FFFFFF';
        }
    }
};
</script>

<template>
    <div class="container mt-4">
        <AdminMenu />
        <h2 class="text-3xl font-bold text-gray-700 mb-6">Detalles de Salida</h2>
        <div v-if="salida" class="bg-white rounded-lg shadow-md p-8">
            <div class="info-section mb-6">
                <p class="text-lg mb-2"><strong class="font-semibold text-gray-600">Guía de Salida:</strong> {{
                    salida.guia_salida }}</p>
                <p class="text-lg mb-2"><strong class="font-semibold text-gray-600">Destino:</strong> {{ salida.destino
                    }}</p>
                <p class="text-lg mb-2"><strong class="font-semibold text-gray-600">Fecha:</strong> {{ salida.fecha ?
                    new Date(salida.fecha).toLocaleDateString() : 'Sin fecha' }}
                </p>
                <p class="text-lg mb-4"><strong class="font-semibold text-gray-600">Observaciones:</strong> {{
                    salida.observaciones || "N/A" }}</p>
            </div>

            <h3 class="text-xl font-semibold text-gray-700 mb-4">Productos</h3>
            <div v-if="salida.items && salida.items.length"
                class="product-list grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                <div v-for="item in salida.items" :key="item.id" class="product-card bg-gray-50 rounded-lg shadow p-4">
                    <p class="font-semibold text-gray-700 mb-2">{{ item.inventario?.producto?.nombre
                        || 'Producto desconocido' }}</p>

                    <div v-if="item.inventario.color" class="mb-2">
                        <span class="color-label inline-block px-3 py-1 rounded-full text-sm font-semibold" :style="{
                            backgroundColor: item.inventario.color.hex || '#FFFFFF',
                            color: getTextColor(item.inventario.color.hex || '#FFFFFF')
                        }">
                            {{ item.inventario.color.descripcion }}
                        </span>
                    </div>
                    <p class="text-gray-600 mb-1">Tamaño: <span class="badge bg-info me-1">
                            {{ item.inventario.tamano ? item.inventario.tamano.descripcion
                                : 'No disponible' }}
                        </span>
                    </p>
                    <p class="text-gray-600 mb-1">Longitud: <span class="badge bg-secondary me-1">
                            {{ item.inventario.longitud ?
                                item.inventario.longitud.descripcion : 'No disponible' }}
                        </span>
                    </p>

                    <p class="text-gray-600">Cantidad: <span class="badge bg-secondary me-1">
                            {{ item.cantidad }}
                        </span> - Precio: <span class="badge bg-secondary me-1">
                            {{ item.precio_unitario }}
                        </span></p>
                </div>
            </div>
            <br>
            <button
                class="bg-blue-600 hover:bg-blue-700 text-white font-medium py-2 px-6 rounded-full mt-6 transition-all duration-300"
                @click="goBack">
                Regresar a Salidas
            </button>
        </div>
        <p v-else class="text-gray-500">Cargando detalles de la salida...</p>
    </div>
</template>

<style scoped>
.container {
    max-width: 800px;
    margin: 0 auto;
}

.info-section p {
    font-size: 1rem;
}

.product-list {
    display: grid;
    gap: 1rem;
}

.product-card {
    background-color: #f9fafb;
    padding: 1rem;
    border-radius: 0.5rem;
    box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.05);
}

.color-label {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    border-radius: 9999px;
    font-size: 0.875rem;
    font-weight: 500;
    text-align: center;
}

button {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background-color: #3b82f6;
    border: none;
    color: white;
    border-radius: 0.5rem;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s ease;
}

button:hover {
    background-color: #2563eb;
}
</style>
