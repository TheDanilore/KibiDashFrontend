<script>
import AdminMenu from './AdminMenu.vue';
import axios from 'axios';
import UsuariosForm from './UsuariosForm.vue';

export default {
    components: {
        AdminMenu,
        UsuariosForm,
    },
    data() {
        return {
            roles: [],
            isDragging: false, // estado para controlar el arrastre
            error: null,
            usuarios: [], // Array para los proveedores
            mostrarFormulario: false,
            usuarioEditado: null,
        };
    },
    methods: {
        async obtenerUsuarios() {
            try {
                const response = await axios.get('/api/usuarios'); // Asegúrate de que la ruta sea correcta
                this.usuarios = response.data;
            } catch (error) {
                console.error('Error al obtener los usuarios:', error);
            }
        },
        async obtenerRoles() {
            try {
                const response = await axios.get('/api/roles');
                this.roles = response.data;
            } catch (error) {
                console.error('Error al obtener roles:', error);
            }
        },

        mostrarFormularioCreacion() {
            this.usuarioEditado = null;
            this.mostrarFormulario = true;
        },
        editarUsuario(usuario) {
            // Asegurarse de que los roles estén en el formato correcto
            this.usuarioEditado = {
                ...usuario,
                roles: usuario.roles.map(rol => rol.name) // Asegurarse de que solo pasamos los nombres
            };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerUsuarios();
            this.cerrarFormulario();
        },
        async eliminarUsuario(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este usuario?')) {
                try {
                    await axios.delete(`/api/usuarios/${id}`);
                    this.obtenerUsuarios();
                } catch (error) {
                    console.error('Error al eliminar el usuario:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.usuarioEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerUsuarios();
        this.obtenerRoles();
    }
};
</script>
<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Usuarios</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Email</th>
                        <th>Roles</th>
                        <th>Avatar</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="usuario in usuarios" :key="usuario.id">
                        <td>{{ usuario.id }}</td>
                        <td>{{ usuario.name }}</td>
                        <td>{{ usuario.email }}</td>
                        <td>
                            <span v-for="(rol, index) in usuario.roles" :key="rol.id">
                                {{ rol.name }}<span v-if="index < usuario.roles.length - 1">, </span>
                            </span>
                        </td>
                        <td>
                            <img :src="`http://localhost:8000/storage/${usuario.avatar}`" alt="Avatar"
                                class="rounded-circle" width="80" height="80">
                        </td>
                        <td>
                            <span v-if="usuario.estado === 'Activo'" class="badge bg-success">Activo</span>
                            <span v-else class="badge bg-danger">Inactivo</span>
                        </td>
                        <td>
                            <button @click="editarUsuario(usuario)" class="btn btn-primary btn-sm">Editar</button>
                            <button @click="eliminarUsuario(usuario.id)" class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Usuario</button>
        </div>

        <!-- Formulario para crear/editar tamaño -->
        <UsuariosForm v-if="mostrarFormulario" :usuarioEditado="usuarioEditado" @guardar="onGuardar"
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

.drop-area {
    border: 2px dashed #007bff;
    border-radius: 5px;
    padding: 20px;
    cursor: pointer;
    text-align: center;
    transition: background-color 0.3s;
    margin-top: 5px;
}

.drop-area.highlight {
    background-color: rgba(0, 123, 255, 0.1);
    /* Color de fondo al arrastrar */
}

.preview {
    max-width: 150px;
    /* Cambia este valor para ajustar el tamaño */
    max-height: 150px;
    /* Cambia este valor para ajustar el tamaño */
    border-radius: 50%;
}
</style>
