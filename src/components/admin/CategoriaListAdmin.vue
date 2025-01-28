<script>
import AdminMenu from './AdminMenu.vue';
import CategoriaForm from './CategoriaForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        CategoriaForm,
    },
    data() {
        return {
            error: null,
            categorias: [],
            mostrarFormulario: false,
            categoriaEditada: null,
        };
    },
    methods: {
        async obtenerCategorias() {
            try {
                const response = await axios.get('/api/categoriaproductos'); // Asegúrate de que la ruta sea correcta
                this.categorias = response.data;
            } catch (error) {
                console.error('Error al obtener las categorías:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.categoriaEditada = null;
            this.mostrarFormulario = true;
        },
        editarCategoria(categoria) {
            this.categoriaEditada = { ...categoria };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerCategorias();
            this.cerrarFormulario();
        },
        async eliminarCategoria(id) {
            if (confirm('¿Estás seguro de que deseas eliminar esta categoria?')) {
                try {
                    await axios.delete(`/api/categoriaproductos/${id}`);
                    this.obtenerCategorias();
                } catch (error) {
                    console.error('Error al eliminar la categoría:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.categoriaEditada = null;
        },

    },

    //cargar listas 
    mounted() {
        this.obtenerCategorias();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Categorías</h2>
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
                    <tr v-for="categoria in categorias" :key="categoria.id">
                        <td>{{ categoria.id }}</td>
                        <td>{{ categoria.descripcion }}</td>
                        <td>
                            <button @click="editarCategoria(categoria)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarCategoria(categoria.id)"
                                class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nueva Categoria</button>
        </div>

        <!-- Formulario para crear/editar categoria -->
        <CategoriaForm v-if="mostrarFormulario" :categoriaEditada="categoriaEditada" @guardar="onGuardar"
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
