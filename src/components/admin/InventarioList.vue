<script>
import AdminMenu from './AdminMenu.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
    },
    data() {
        return {
            inventario: [],
        };
    },
    methods: {
        async obtenerInventario() {
            try {
                const response = await axios.get('/api/inventario/producto');
                this.inventario = response.data;
            } catch (error) {
                console.error('Error al cargar el inventario:', error);
            }
        },
    },
    created() {
        this.obtenerInventario();
    },
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <div>
            <h1>Inventario</h1>

            <div class="table-responsive">
                <table class="table table-striped table-bordered text-center">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Producto</th>
                            <th>Color</th>
                            <th>Tamaño</th>
                            <th>Longitud</th>
                            <th>Precio Unitario</th>
                            <th>Cantidad</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="item in inventario" :key="item.id">
                            <td>{{ item.id }}</td>
                            <td>{{ item.producto.nombre }}</td>
                            <td>
                                <template v-if="item.color?.descripcion != null">
                                    <h6><span class="badge bg-secondary me-1">
                                            {{ item.color?.descripcion || 'N/A' }}
                                        </span>
                                    </h6>
                                </template>
                                <span v-else>-</span>
                            </td>
                            <td>
                                <template v-if="item.tamano?.descripcion != null">
                                    <h6><span class="badge bg-info me-1">
                                            {{ item.tamano?.descripcion || 'N/A' }}
                                        </span>
                                    </h6>
                                </template>
                                <span v-else>-</span>
                            </td>
                            <td>
                                <h6>
                                    <template v-if="item.longitud?.descripcion != null">
                                        <span class="badge bg-info me-1">
                                            {{ item.longitud?.descripcion || 'N/A' }}
                                        </span>
                                    </template>
                                    <span v-else>-</span>
                                </h6>
                            </td>
                            <td>S/{{ item.precio_unitario.toFixed(2) || 0 }}</td>
                            <td>
                                <template v-if="item.cantidad >= 500">
                                    <span class="badge bg-success me-1">
                                        {{ item.cantidad }}
                                    </span>
                                </template>
                                <template v-if="item.cantidad < 500 && item.cantidad > 50">
                                    <span class="badge bg-primary me-1">
                                        {{ item.cantidad }}
                                    </span>
                                </template>
                                <template v-if="item.cantidad <= 50 && item.cantidad > 10">
                                    <span class="badge bg-warning me-1">
                                        {{ item.cantidad }}
                                    </span>
                                </template>
                                <template v-if="item.cantidad <= 10">
                                    <span class="badge bg-danger me-1">
                                        {{ item.cantidad }}
                                    </span>
                                </template>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>


<style scoped>
table {
    width: 100%;
    border-collapse: collapse;
}

th,
td {
    padding: 10px;
    border: 1px solid #ddd;
    text-align: left;
}
</style>
