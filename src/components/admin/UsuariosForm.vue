<script>
import axios from 'axios';

export default {
    props: {
        usuarioEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            modo: 'crear',
            error: null,
            errores: {
                name: '',
                email: '',
                password: '',
                password_confirmation: '',
                avatar: '',
                estado: '',
                roles: ''
            },
            usuario: {
                name: '',
                email: '',
                password: '',
                password_confirmation: '',
                avatar: null,
                avatarUrl: '', // URL actual del avatar
                avatarPreview: '',
                estado: 'Activo',
                roles: []
            },
            validaciones: {
                name: {
                    minLength: 5,
                    maxLength: 150,
                    pattern: /^[A-Za-z0-9\s.,#-nñ]+$/
                },
                email: {
                    minLength: 5,
                    maxLength: 150,
                    pattern: /^[a-zA-Z0-9._-nñ]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}$/
                },
                password: {
                    minLength: 5,
                    maxLength: 50,
                    pattern: /^[A-Za-z0-9\-+@.ñÑ]+$/
                }
            },
            roles: [],
            isDragging: false,
        };
    },
    watch: {
        usuarioEditado: {
            immediate: true,
            handler(newValue) {
                console.log('Usuario editado recibido:', newValue);
                if (newValue) {
                    this.modo = 'editar';
                    this.usuario = {
                        id: newValue.id, // Importante: incluir el ID
                        name: newValue.name,
                        email: newValue.email,
                        password: '',
                        password_confirmation: '',
                        estado: newValue.estado || 'Activo',
                        avatar: null, // El archivo actual
                        avatarUrl: newValue.avatar ? `http://localhost:8000/storage/${newValue.avatar}` : '',
                        avatarPreview: newValue.avatar ? `http://localhost:8000/storage/${newValue.avatar}` : '',
                        roles: Array.isArray(newValue.roles)
                            ? newValue.roles.map(rol => typeof rol === 'object' ? rol.name : rol)
                            : []
                    };
                    console.log('Usuario procesado:', this.usuario);
                }
            }
        }
    },
    computed: {
        esFormularioValido() {
            // Validar campos básicos
            const validaciones = [
                this.validarName(this.usuario.name),
                this.validarEmail(this.usuario.email)
            ];
            // Solo validar contraseña en modo crear o si se ha ingresado una nueva contraseña en modo editar
            if (this.modo === 'crear' || this.usuario.password) {
                validaciones.push(
                    this.validarPassword(this.usuario.password),
                    this.usuario.password === this.usuario.password_confirmation
                );
            }

            return validaciones.every(v => v === true);
        },
        tituloFormulario() {
            return this.modo === 'editar' ? 'Editar Usuario' : 'Registrar Usuario';
        }

    },
    methods: {
        validarName(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.name;

            if (!valor || valor.length < minLength) {
                this.errores.name = `El nombre debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.name = `El nombre no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.name = 'El nombre contiene caracteres no válidos';
                return false;
            }

            this.errores.name = '';
            return true;
        },
        validarEmail(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.email;

            if (!valor || valor.length < minLength) {
                this.errores.email = `El correo electrónico debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.email = `El correo electrónico no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.email = 'El correo electrónico contiene caracteres no válidos';
                return false;
            }

            this.errores.email = '';
            return true;
        },
        validarPassword(valor) {
            if (this.modo === 'editar' && !valor) {
                return true; // Permitir contraseña vacía en modo edición
            }

            const { minLength, maxLength, pattern } = this.validaciones.password;

            if (!valor || valor.length < minLength) {
                this.errores.password = `La contraseña debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.password = `La contraseña no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.password = 'La contraseña contiene caracteres no válidos';
                return false;
            }

            this.errores.password = '';
            return true;
        },
        async obtenerRoles() {
            try {
                const response = await axios.get('/api/roles');
                this.roles = response.data;
            } catch (error) {
                console.error('Error al obtener roles:', error);
            }
        },
        async obtenerAvatarUsuario(usuarioId) {
            try {
                const response = await axios.get(`/api/usuarios/${usuarioId}/avatar`);
                if (response.data && response.data.url) {
                    this.usuario.avatarUrl = response.data.url;
                    this.usuario.avatarPreview = response.data.url;
                }
            } catch (error) {
                console.error('Error al obtener avatar:', error);
            }
        },

        handleFileUpload(event) {
            const file = event.target.files[0];
            this.procesarArchivo(file);
        },

        handleDrop(event) {
            event.preventDefault();
            const file = event.dataTransfer.files[0];
            this.procesarArchivo(file);
            this.isDragging = false;
        },

        procesarArchivo(file) {
            if (file && file.type.startsWith('image/')) {
                this.usuario.avatar = file;
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.usuario.avatarPreview = e.target.result;
                };
                reader.readAsDataURL(file);
                this.errores.avatar = '';
            } else {
                this.errores.avatar = 'Por favor, seleccione un archivo de imagen válido';
            }
        },
        toggleEstado() {
            this.usuario.estado = this.usuario.estado === 'Activo' ? 'Desactivado' : 'Activo';
        },
        async guardarUsuario() {
            try {
                const formData = new FormData();
                formData.append('name', this.usuario.name);
                formData.append('email', this.usuario.email);
                formData.append('estado', this.usuario.estado);

                // Agregar roles
                this.usuario.roles.forEach(rol => {
                    formData.append('roles[]', rol);
                });

                // Solo agregar contraseña si se está creando o se ha modificado
                if (this.modo === 'crear' || this.usuario.password) {
                    formData.append('password', this.usuario.password);
                    formData.append('password_confirmation', this.usuario.password_confirmation);
                }

                // Solo agregar avatar si hay uno nuevo
                if (this.usuario.avatar instanceof File) {
                    formData.append('avatar', this.usuario.avatar);
                }

                const url = this.modo === 'editar'
                    ? `/api/usuarios/${this.usuario.id}`
                    : '/api/usuarios';

                const method = this.modo === 'editar' ? 'post' : 'post';

                if (this.modo === 'editar') {
                    formData.append('_method', 'PUT');
                }

                const response = await axios({
                    method: method,
                    url: url,
                    data: formData,
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                });

                this.$emit('guardar', response.data);
            } catch (error) {
                console.error('Error al guardar usuario:', error);
                this.error = 'Error al guardar el usuario';
            }
        },

    },
    mounted() {
        // Añadido: Cargar roles al montar el componente
        this.obtenerRoles();
    },
};
</script>

<template>
    <div class="container mt-0">
        <h2 class="mb-4 text-center">{{ usuarioEditado ? 'Editar' : 'Registrar' }} Usuario</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>
        <!-- Formulario para crear o editar un usuario -->
        <form @submit.prevent="guardarUsuario" class="form-container">
            <div class="form-group mb-3">
                <label for="name">Nombre</label>
                <input v-model="usuario.name" type="text" id="name" class="form-control"
                    :class="{ 'is-invalid': errores.name }" required>
                <div class="invalid-feedback" v-if="errores.name">
                    {{ errores.name }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="email">Email</label>
                <input v-model="usuario.email" type="email" id="email" class="form-control"
                    :class="{ 'is-invalid': errores.email }" required>
                <div class="invalid-feedback" v-if="errores.email">
                    {{ errores.email }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="password">contraseña</label>
                <input v-model="usuario.password" type="password" id="password" class="form-control"
                    :class="{ 'is-invalid': errores.password }">
                <div class="invalid-feedback" v-if="errores.password">
                    {{ errores.password }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="password_confirmation">Confirmar contraseña</label>
                <input v-model="usuario.password_confirmation" type="password" id="password_confirmation"
                    class="form-control" :class="{ 'is-invalid': errores.password_confirmation }">
                <div class="invalid-feedback" v-if="errores.password_confirmation">
                    {{ errores.password_confirmation }}
                </div>
            </div>

            <!-- Área de arrastre para el avatar con preview -->
            <div class="form-group mb-3" @dragover.prevent @drop.prevent="handleDrop">
                <label for="avatar">Avatar</label>
                <div class="drop-area" :class="{ 'highlight': isDragging }" @dragenter="isDragging = true"
                    @dragleave="isDragging = false" @dragover.prevent @click="$refs.avatarInput.click()">
                    <template v-if="usuario.avatarPreview || usuario.avatarUrl">
                        <img :src="usuario.avatarPreview || usuario.avatarUrl" alt="Avatar Preview"
                            class="preview img-fluid" style="max-width: 200px; max-height: 200px;">
                    </template>
                    <p v-else>Arrastra y suelta una imagen aquí o haz clic para seleccionar</p>
                </div>
                <input type="file" ref="avatarInput" @change="handleFileUpload" accept="image/*" class="d-none">
                <div class="invalid-feedback" v-if="errores.avatar">
                    {{ errores.avatar }}
                </div>
            </div>

            <div class="form-group mb-3">
                <label for="roles">Roles</label>
                <select v-model="usuario.roles" multiple class="form-control">
                    <option v-for="rol in roles" :key="rol.id" :value="rol.name">
                        {{ rol.name }}
                    </option>
                </select>
            </div>
            <div class="form-group mb-3">
                <label for="estadoToggle">Estado</label>
                <div class="form-check form-switch">
                    <input type="checkbox" class="form-check-input" id="estadoToggle"
                        :checked="usuario.estado === 'Activo'" @change="toggleEstado">
                    <label class="form-check-label" for="estadoToggle">{{ usuario.estado }}</label>
                </div>
            </div>
            <br>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ usuarioEditado ? 'Actualizar' : 'Registrar' }} Usuario
                </button>
            </div>
        </form>
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
