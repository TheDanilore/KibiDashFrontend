<template>
    <div class="container mt-0">
        <AdminMenu />
        <div>
            <h1>Movimientos de Inventario</h1>
            <div class="table-responsive">
                <table class="table table-striped table-bordered text-center">
                    <thead>
                        <tr>
                            <th>Inventario ID</th>
                            <th>Producto</th>
                            <th>Tipo de Movimiento</th>
                            <th>Cantidad</th>
                            <th>Stock Anterior</th>
                            <th>Fecha</th>
                            <th>Usuario</th>
                            <th>Observaciones</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="movimiento in movimientos" :key="movimiento.id">
                            <td>{{ movimiento.inventario_id }}</td>
                            <td>{{ movimiento.inventario.producto.nombre }}</td>
                            <td>
                                <template v-if="movimiento.tipo_movimiento == 'entrada'">
                                    <h5>
                                        <span class="badge bg-success">
                                            {{ movimiento.tipo_movimiento }}
                                        </span>
                                    </h5>
                                </template>
                                <template v-if="movimiento.tipo_movimiento == 'salida'">
                                    <h5>
                                        <span class="badge bg-danger">
                                            {{ movimiento.tipo_movimiento }}
                                        </span>
                                    </h5>
                                </template>
                                <template v-if="movimiento.tipo_movimiento == 'ajuste'">
                                    <h5>
                                        <span class="badge bg-info">
                                            {{ movimiento.tipo_movimiento }}
                                        </span>
                                    </h5>
                                </template>
                                <template v-if="movimiento.tipo_movimiento == 'reversion salida'">
                                    <h5>
                                        <span class="badge bg-secondary">
                                            {{ movimiento.tipo_movimiento }}
                                        </span>
                                    </h5>
                                </template>
                                <template v-if="movimiento.tipo_movimiento == 'reversion entrada'">
                                    <h5>
                                        <span class="badge bg-secondary">
                                            {{ movimiento.tipo_movimiento }}
                                        </span>
                                    </h5>
                                </template>
                            </td>
                            <td>{{ movimiento.cantidad }}</td>
                            <td>

                                {{ movimiento.stock_anterior }}
                            </td>
                            <td>{{ new Date(movimiento.fecha).toLocaleDateString() }}</td>
                            <td>{{ movimiento.usuario.name }}</td>
                            <td>{{ movimiento.observaciones }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script>
import AdminMenu from './AdminMenu.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
    },
    data() {
        return {
            movimientos: [],
        };
    },
    methods: {
        async obtenerMovimientos() {
            try {
                const response = await axios.get('/api/movimientosinventario');
                this.movimientos = response.data;
            } catch (error) {
                console.error('Error al cargar los movimientos:', error);
            }
        },
    },
    created() {
        this.obtenerMovimientos();
    },
};
</script>

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
