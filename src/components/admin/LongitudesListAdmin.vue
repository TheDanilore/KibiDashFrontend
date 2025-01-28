<script>
import AdminMenu from './AdminMenu.vue';
import LongitudesForm from './LongitudesForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        LongitudesForm,
    },
    data() {
        return {
            error: null,
            longitudes: [],
            mostrarFormulario: false,
            colorEditada: null,
        };
    },
    methods: {
        async obtenerLongitudes() {
            try {
                const response = await axios.get('/api/longitudes'); // Asegúrate de que la ruta sea correcta
                this.longitudes = response.data;
            } catch (error) {
                console.error('Error al obtener las longitudes:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.longitudEditado = null;
            this.mostrarFormulario = true;
        },
        editarLongitud(longitud) {
            this.longitudEditado = { ...longitud };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerLongitudes();
            this.cerrarFormulario();
        },
        async eliminarLongitud(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta longitud?')) {
                try {
                    await axios.delete(`/api/longitudes/${id}`);
                    this.obtenerLongitudes();
                } catch (error) {
                    console.error('Error al eliminar la longitud:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.longitudEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerLongitudes();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Longitudes</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Descripción</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="longitud in longitudes" :key="longitud.id">
                        <td>{{ longitud.id }}</td>
                        <td>{{ longitud.descripcion }}</td>
                        <td>
                            <button @click="editarLongitud(longitud)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarLongitud(longitud.id)"
                                class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Longitud</button>
        </div>

        <!-- Formulario para crear/editar categoria -->
        <LongitudesForm v-if="mostrarFormulario" :longitudEditado="longitudEditado" @guardar="onGuardar"
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
