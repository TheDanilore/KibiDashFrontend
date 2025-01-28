<script>
import AdminMenu from './AdminMenu.vue';
import UnidadesMedidasForm from './UnidadesMedidasForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        UnidadesMedidasForm,
    },
    data() {
        return {
            error: null,
            unidades: [], // Array para los proveedores
            mostrarFormulario: false,
            unidadEditado: null,
        };
    },
    methods: {
        async obtenerUnidades() {
            try {
                const response = await axios.get('/api/unidadesmedidas'); // Asegúrate de que la ruta sea correcta
                this.unidades = response.data;
            } catch (error) {
                console.error('Error al obtener las unidades de medida:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.unidadEditado = null;
            this.mostrarFormulario = true;
        },
        editarUnidad(unidad) {
            this.unidadEditado = { ...unidad };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerUnidades();
            this.cerrarFormulario();
        },
        async eliminarUnidad(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta unidad de medida?')) {
                try {
                    await axios.delete(`/api/unidadesmedidas/${id}`);
                    this.obtenerUnidades();
                } catch (error) {
                    console.error('Error al eliminar la unidad de medida:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.unidadEditado = null;
        },

    },

    //cargar listas 
    mounted() {
        this.obtenerUnidades();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Unidades de Medidas</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Descripción</th>
                        <th>Abreviatura</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="unidad in unidades" :key="unidad.id">
                        <td>{{ unidad.id }}</td>
                        <td>{{ unidad.descripcion }}</td>
                        <td>{{ unidad.abreviatura }}</td>
                        <td>
                            <button @click="editarUnidad(unidad)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarUnidad(unidad.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Unidad de
                Medida</button>
        </div>

        <!-- Formulario para crear/editar tamaño -->
        <UnidadesMedidasForm v-if="mostrarFormulario" :unidadEditado="unidadEditado" @guardar="onGuardar"
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
