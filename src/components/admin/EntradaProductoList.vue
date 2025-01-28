<script>
import AdminMenu from './AdminMenu.vue';
import EntradaProductoForm from './EntradaProductoForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        EntradaProductoForm,
    },
    data() {
        return {
            error: null,
            entradas: [],
            productos: [],
            proveedores: [],
            mostrarFormulario: false,
            entradaEditada: null,
            salidaSeleccionada: null,
            paginaActual: 1,
            ultimaPagina: 1,
        };
    },
    methods: {
        async obtenerEntradas(pagina = 1) {
            try {
                const response = await axios.get(`/api/entradasproductos?page=${pagina}`);
                console.log('Entradas obtenidas:', response.data);
                if (response.data && response.data.data) {
                    this.entradas = response.data.data;
                    this.paginaActual = response.data.current_page;
                    this.ultimaPagina = response.data.last_page;
                } else {
                    this.entradas = [];
                }
            } catch (error) {
                console.error('Error al obtener las entradas:', error);
            }
        },
        async verDetalles(id) {
            try {
                const response = await axios.get(`/api/entradasproductos/${id}`);
                this.$router.push({ name: 'EntradaProductoDetalles', params: { id: id }, query: { entrada: JSON.stringify(response.data) } });
            } catch (error) {
                console.error("Error al cargar los detalles de la entrada:", error);
                // Agrega aquí un mensaje de error o una acción alternativa
            }
        },
        mostrarFormularioCreacion() {
            this.entradaEditada = null;
            this.mostrarFormulario = true;
        },
        editarEntrada(entrada) {
            this.entradaEditada = { ...entrada };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerEntradas();
            this.cerrarFormulario();
        },
        async eliminarEntrada(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta entrada?')) {
                try {
                    await axios.delete(`/api/entradasproductos/${id}`);
                    await this.obtenerEntradas();
                } catch (error) {
                    console.error('Error al eliminar la entrada:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.entradaEditada = null;
        },

        getTotalProductos(entrada) {
            return entrada.items?.reduce((total, item) => total + item.cantidad, 0) || 0;
        },

        getNombresProductos(entrada) {
            return entrada.items?.map(item => item.inventario.producto.nombre).join(', ') || 'Sin productos';
        }
    },
    mounted() {
        this.obtenerEntradas();
    },
};
</script>

<template>
    <div class="container mt-4">
        <AdminMenu />
        <h2 class="mb-4 text-center">Lista de Entradas de Producto</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Guía de Remisión</th>
                        <th>Producto</th>
                        <th>Cantidad Total</th>
                        <th>Tipo Entrada</th>
                        <th>Fecha de Entrada</th>
                        <th>Proveedor</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody v-if="entradas.length">
                    <tr v-for="entrada in entradas.filter(e => e !== null)" :key="entrada.id">
                        <td>{{ entrada.id }}</td>
                        <td>{{ entrada.guia_remision }}</td>
                        <td>{{ getNombresProductos(entrada) }}</td>
                        <td>{{ getTotalProductos(entrada) }}</td>
                        <td>{{ entrada.tipo_entrada }}</td>
                        <td>{{ entrada.fecha ? new Date(entrada.fecha).toLocaleDateString() : 'Sin fecha' }}
                        </td>
                        <td>{{ entrada.proveedor ? entrada.proveedor.razon_social : 'Sin proveedor' }}</td>
                        <td>
                            <button @click="verDetalles(entrada.id)" class="btn btn-primary btn-sm">Ver
                                Detalles</button>
                            <button @click="eliminarEntrada(entrada.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>

                </tbody>
                <p v-else>No hay entradas disponibles.</p>
            </table>
        </div>
        <!-- Paginación -->
        <button @click="obtenerEntradas(paginaActual - 1)" :disabled="paginaActual === 1">Anterior</button>
        <button @click="obtenerEntradas(paginaActual + 1)" :disabled="paginaActual === ultimaPagina">Siguiente</button>

        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Entrada</button>
        </div>

        <!-- Formulario para crear/editar entrada -->
        <EntradaProductoForm v-if="mostrarFormulario" :entradaEditada="entradaEditada" :productos="productos"
            :proveedores="proveedores" @guardar="onGuardar" @cerrar="cerrarFormulario" />

    </div>
</template>
