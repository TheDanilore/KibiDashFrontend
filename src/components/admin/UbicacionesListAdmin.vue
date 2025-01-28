<script>
import AdminMenu from './AdminMenu.vue';
import UbicacionesForm from './UbicacionesForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        UbicacionesForm,
    },
    data() {
        return {
            error: null,
            ubicaciones: [],
            mostrarFormulario: false,
            ubicacionEditado: null,
        };
    },
    methods: {
        async obtenerUbicaciones() {
            try {
                const response = await axios.get('/api/ubicaciones'); // Asegúrate de que la ruta sea correcta
                this.ubicaciones = response.data;
            } catch (error) {
                console.error('Error al obtener las ubicaciones:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.ubicacionEditado = null;
            this.mostrarFormulario = true;
        },
        editarUbicacion(ubicacion) {
            this.ubicacionEditado = { ...ubicacion };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerUbicaciones();
            this.cerrarFormulario();
        },
        async eliminarUbicacion(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta ubicación?')) {
                try {
                    await axios.delete(`/api/ubicaciones/${id}`);
                    this.obtenerProveedores();
                } catch (error) {
                    console.error('Error al eliminar la ubicación:', error.response ? error.response.data : error.message);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.ubicacionEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerUbicaciones();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Ubicaciones</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Descripción</th>
                        <th>Código</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="ubicacion in ubicaciones" :key="ubicacion.id">
                        <td>{{ ubicacion.id }}</td>
                        <td>{{ ubicacion.descripcion }}</td>
                        <td>{{ ubicacion.codigo }}</td>
                        <td>
                            <button @click="editarUbicacion(ubicacion)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarUnidad(ubicacion.id)"
                                class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Ubicación</button>
        </div>


        <!-- Formulario para crear/editar tamaño -->
        <UbicacionesForm v-if="mostrarFormulario" :ubicacionEditado="ubicacionEditado" @guardar="onGuardar"
            @cerrar="cerrarFormulario" />

    </div>
</template>



<style scoped>
.container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    text-align: center;
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
