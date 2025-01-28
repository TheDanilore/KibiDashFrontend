<script>
import AdminMenu from './AdminMenu.vue';
import RolesForm from './RolesForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        RolesForm,
    },
    data() {
        return {
            error: null,
            roles: [],
            permisos: [], // Array para los proveedores
            mostrarFormulario: false,
            proveedorEditado: null,
        };
    },
    methods: {
        async obtenerRoles() {
            try {
                const response = await axios.get('/api/roles');
                this.roles = response.data;
            } catch (error) {
                console.error('Error al obtener roles:', error);
            }
        },
        async obtenerPermisos() {
            try {
                const response = await axios.get('/api/permissions'); // Asegúrate de tener una API para obtener permisos
                this.permisos = response.data;
            } catch (error) {
                console.error('Error al obtener permisos:', error);
            }
        },
        mostrarFormularioCreacion() {
            this.rolEditado = null;
            this.mostrarFormulario = true;
        },
        editarRol(rol) {
            this.rolEditado = { ...rol };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerRoles();
            await this.obtenerPermisos();
            this.cerrarFormulario();
        },
        async eliminarRol(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este rol?')) {
                try {
                    await axios.delete(`/api/roles/${id}`);
                    this.obtenerRoles();
                    this.obtenerPermisos();
                } catch (error) {
                    console.error('Error al eliminar el permiso:', error.response ? error.response.data : error.message);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.rolEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerRoles();
        this.obtenerPermisos();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Roles</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Guard Name</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="rol in roles" :key="rol.id">
                        <td>{{ rol.id }}</td>
                        <td>{{ rol.name }}</td>
                        <td>{{ rol.guard_name }}</td>
                        <td>
                            <span v-if="rol.estado === 'Activo'" class="badge bg-success">Activo</span>
                            <span v-else class="badge bg-danger">Inactivo</span>
                        </td>
                        <td>
                            <button @click="editarRol(rol)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarRol(rol.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Rol</button>
        </div>

        <!-- Formulario para crear/editar permiso -->
        <RolesForm v-if="mostrarFormulario" :rolEditado="rolEditado" @guardar="onGuardar" @cerrar="cerrarFormulario" />
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
