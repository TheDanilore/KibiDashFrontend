<script>
import AdminMenu from './AdminMenu.vue';
import ProveedoresForm from './ProveedoresForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        ProveedoresForm,
    },
    data() {
        return {
            error: null,
            proveedores: [], // Array para los proveedores
            mostrarFormulario: false,
            proveedorEditado: null,
        };
    },
    methods: {
        async obtenerProveedores() {
            try {
                const response = await axios.get('/api/proveedores'); // Asegúrate de que la ruta sea correcta
                this.proveedores = response.data;
            } catch (error) {
                console.error('Error al obtener las proveedores:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.proveedorEditado = null;
            this.mostrarFormulario = true;
        },
        editarProveedor(proveedor) {
            this.proveedorEditado = { ...proveedor };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerProveedores();
            this.cerrarFormulario();
        },
        async eliminarProveedor(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este proveedor?')) {
                try {
                    await axios.delete(`/api/proveedores/${id}`);
                    this.obtenerProveedores();
                } catch (error) {
                    console.error('Error al eliminar el permiso:', error.response ? error.response.data : error.message);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.proveedorEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerProveedores();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Proveedores</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>RUC</th>
                        <th>Razon Social</th>
                        <th>Dirección</th>
                        <th>Telélefono</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="proveedor in proveedores" :key="proveedor.id">
                        <td>{{ proveedor.id }}</td>
                        <td>{{ proveedor.ruc }}</td>
                        <td>{{ proveedor.razon_social }}</td>
                        <td>{{ proveedor.direccion }}</td>
                        <td>{{ proveedor.telefono }}</td>
                        <td>
                            <span v-if="proveedor.estado === 'Activo'" class="badge bg-success">Activo</span>
                            <span v-else class="badge bg-danger">Inactivo</span>
                        </td>
                        <td>
                            <button @click="editarProveedor(proveedor)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarCategoria(proveedor.id)"
                                class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Proveedor</button>
        </div>

        <!-- Formulario para crear/editar permiso -->
        <ProveedoresForm v-if="mostrarFormulario" :proveedorEditado="proveedorEditado" @guardar="onGuardar"
            @cerrar="cerrarFormulario" />


    </div>
</template>


<style scoped>
.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    /* Centra todo el contenido dentro del contenedor */
}

h2 {
    font-size: 1.8rem;
    margin-bottom: 20px;
}

.form-container {
    background-color: #f8f9fa;
    padding: 1.5rem;
    border-radius: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    margin-top: 20px;
}

.table {
    margin-top: 1.5rem;
}

.table th,
.table td {
    vertical-align: middle;
    text-align: center;
    /* Centra el contenido de las celdas */
}

.btn {
    margin: 0 5px;
    /* Espacio entre los botones */
}

.mt-4 {
    margin-top: 1.5rem;
}
</style>
