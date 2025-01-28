<script>
import AdminMenu from './AdminMenu.vue';
import SalidaProductoForm from './SalidaProductoForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        SalidaProductoForm,
    },
    data() {
        return {
            salidas: [],
            productos: [],
            mostrarFormulario: false,
            salidaEditada: null,
            salidaSeleccionada: null,
            paginaActual: 1,
            ultimaPagina: 1,
        };
    },
    methods: {
        async obtenerSalidas(pagina = 1) {
            try {
                const response = await axios.get(`/api/salidasproductos?page=${pagina}`);
                console.log('Salidas obtenidas:', response.data);
                if (response.data && response.data.data) {
                    this.salidas = response.data.data;
                    this.paginaActual = response.data.current_page;
                    this.ultimaPagina = response.data.last_page;
                } else {
                    this.salidas = [];
                }
            } catch (error) {
                console.error('Error al obtener las salidas:', error);
            }
        },
        async verDetalles(id) {
            try {
                const response = await axios.get(`/api/salidasproductos/${id}`);
                this.$router.push({ name: 'SalidaProductoDetalles', params: { id: id }, query: { salida: JSON.stringify(response.data) } });
            } catch (error) {
                console.error("Error al cargar los detalles de la salida:", error);
                // Agrega aquí un mensaje de error o una acción alternativa
            }
        },
        mostrarFormularioCreacion() {
            this.salidaEditada = null;
            this.mostrarFormulario = true;
        },
        editarSalida(salida) {
            this.salidaEditada = { ...salida };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerSalidas();
            this.cerrarFormulario();
        },
        async eliminarSalida(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta salida?')) {
                try {
                    await axios.delete(`/api/salidasproductos/${id}`);
                    this.obtenerSalidas();
                } catch (error) {
                    console.error('Error al eliminar la salida:', error);
                }
            }
        },
        getColoresProductos(salida) {
            return salida.items?.map(item => item.inventario.color.descripcion).join(', ') || 'Sin color';
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
        },
        getTotalProductos(salida) {
            return salida.items?.reduce((total, item) => total + item.cantidad, 0) || 0;
        },

        getNombresProductos(salida) {
            return salida.items?.map(item => item.inventario.producto.nombre).join(', ') || 'Sin productos';
        }

    },
    mounted() {
        this.obtenerSalidas();
    },
};
</script>

<template>
    <div class="container mt-4">
        <AdminMenu />
        <h2 class="mb-4 text-center">Lista de Salidas de Producto</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Guía de Salida</th>
                        <th>Producto</th>
                        <th>Cantidad Total</th>
                        <th>Tipo Salida</th>
                        <th>Fecha de Salida</th>
                        <th>Destino</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody v-if="salidas.length">
                    <tr v-for="salida in salidas.filter(e => e !== null)" :key="salida.id">
                        <td>{{ salida.id }}</td>
                        <td>{{ salida.guia_salida }}</td>
                        <td>{{ getNombresProductos(salida) }}</td>
                        <td>{{ getTotalProductos(salida) }}</td>
                        <td>{{ salida.tipo_salida }}</td>
                        <td>{{ salida.fecha ? new Date(salida.fecha).toLocaleDateString() : 'Sin fecha' }}
                        </td>
                        <td>{{ salida.destino }}</td>
                        <td>
                            <button @click="verDetalles(salida.id)" class="btn btn-primary btn-sm">Ver Detalles</button>
                            <button @click="eliminarSalida(salida.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
                <p v-else>No hay salidas disponibles.</p>
            </table>

        </div>
        <!-- Paginación -->
        <button @click="obtenerSalidas(paginaActual - 1)" :disabled="paginaActual === 1">Anterior</button>
        <button @click="obtenerSalidas(paginaActual + 1)" :disabled="paginaActual === ultimaPagina">Siguiente</button>

        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Salida</button>
        </div>

        <!-- Formulario para crear/editar entrada -->
        <SalidaProductoForm v-if="mostrarFormulario" :salidaEditada="salidaEditada" :productos="productos"
            @guardar="onGuardar" @cerrar="cerrarFormulario" />


    </div>
</template>
