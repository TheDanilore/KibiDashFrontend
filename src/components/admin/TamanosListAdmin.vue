<script>
import AdminMenu from './AdminMenu.vue';
import TamanosForm from './TamanosForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        TamanosForm,
    },
    data() {
        return {
            error: null,
            tamanos: [],
            mostrarFormulario: false,
            tamanoEditado: null,
        };
    },
    methods: {
        async obtenerTamanos() {
            try {
                const response = await axios.get('/api/tamanos'); // Asegúrate de que la ruta sea correcta
                this.tamanos = response.data;
            } catch (error) {
                console.error('Error al obtener los tamaños:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.tamanoEditado = null;
            this.mostrarFormulario = true;
        },
        editarTamano(tamano) {
            this.tamanoEditado = { ...tamano };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerTamanos();
            this.cerrarFormulario();
        },
        async eliminarTamano(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este tamaño?')) {
                try {
                    await axios.delete(`/api/tamanos/${id}`);
                    this.obtenerProveedores();
                } catch (error) {
                    console.error('Error al eliminar el tamaño:', error.response ? error.response.data : error.message);
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
        this.obtenerTamanos();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Tamaños</h2>
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
                    <tr v-for="tamano in tamanos" :key="tamano.id">
                        <td>{{ tamano.id }}</td>
                        <td>{{ tamano.descripcion }}</td>
                        <td>
                            <button @click="editarTamano(tamano)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarTamano(tamano.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>

        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Tamaño</button>
        </div>

        <!-- Formulario para crear/editar tamaño -->
        <TamanosForm v-if="mostrarFormulario" :tamanoEditado="tamanoEditado" @guardar="onGuardar"
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
