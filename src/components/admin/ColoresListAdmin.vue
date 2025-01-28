<script>
import AdminMenu from './AdminMenu.vue';
import ColoresForm from './ColoresForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        ColoresForm,
    },
    data() {
        return {
            error: null,
            colores: [],
            mostrarFormulario: false,
            colorEditada: null,
        };
    },
    methods: {
        async obtenerColores() {
            try {
                const response = await axios.get('/api/colores'); // Asegúrate de que la ruta sea correcta
                this.colores = response.data;
            } catch (error) {
                console.error('Error al obtener los colores:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.colorEditado = null;
            this.mostrarFormulario = true;
        },
        editarColor(color) {
            this.colorEditado = { ...color };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerColores();
            this.cerrarFormulario();
        },
        async eliminarColor(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este color?')) {
                try {
                    await axios.delete(`/api/colores/${id}`);
                    this.obtenerColores();
                } catch (error) {
                    console.error('Error al eliminar el color:', error);
                }
            }
        },
        getTextColor(hexColor) {
            const r = parseInt(hexColor.substr(1, 2), 16);
            const g = parseInt(hexColor.substr(3, 2), 16);
            const b = parseInt(hexColor.substr(5, 2), 16);
            const brightness = (r * 299 + g * 587 + b * 114) / 1000;
            return brightness > 128 ? '#000000' : '#FFFFFF';
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.colorEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerColores();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Colores</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Descripción</th>
                        <th>Hexadecimal</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="color in colores" :key="color.id">
                        <td>{{ color.id }}</td>
                        <td>
                            <span class="color-label inline-block px-3 py-1 rounded-full text-sm font-semibold" :style="{
                                backgroundColor: color.hex || '#FFFFFF',
                                color: getTextColor(color.hex || '#FFFFFF')
                            }">
                                {{ color.descripcion }}
                            </span>
                            <!-- <template v-if="color.hex != null">
                                <span class="badge" :style="{
                                    backgroundColor: color.hex || '#FFFFFF',
                                    color: getTextColor(color.hex || '#FFFFFF')
                                }">
                                    {{ color.descripcion }}
                                </span>
                            </template>
<span v-else>{{ color.descripcion }}</span> -->
                        </td>
                        <td>{{ color.hex }}</td>
                        <td>
                            <button @click="editarColor(color)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarColor(color.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Color</button>
        </div>

        <!-- Formulario para crear/editar categoria -->
        <ColoresForm v-if="mostrarFormulario" :colorEditado="colorEditado" @guardar="onGuardar"
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
