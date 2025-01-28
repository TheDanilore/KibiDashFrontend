<script>
import axios from 'axios';

export default {
    props: {
        rolEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            permisos: [],
            error: null,
            errores: {
                name: '',
                guard_name: '',
                estado: '',
                permissions: '',
            },
            rol: {
                name: '',
                guard_name: 'web',
                estado: 'Activo',
                permissions: [],
            },
            permisosSeleccionados: [], // Array para mantener los permisos seleccionados
            validaciones: {
                name: {
                    minLength: 3,
                    maxLength: 30,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
                guard_name: {
                    minLength: 3,
                    maxLength: 20,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
            }
        };
    },
    computed: {
        esFormularioValido() {
            return this.validarName(this.rol.name) &&
                this.validarGuardName(this.rol.guard_name) &&
                this.validarEstado(this.rol.estado) &&
                this.validarPermisos(this.permisosSeleccionados);
        },
        // Agregar computed property para agrupar permisos por módulo
        permisosPorModulo() {
            const grupos = {};
            this.permisos.forEach(permiso => {
                // Asumiendo que los permisos siguen el formato 'modulo.accion'
                const [modulo] = permiso.name.split('.');
                if (!grupos[modulo]) {
                    grupos[modulo] = [];
                }
                grupos[modulo].push(permiso);
            });
            return grupos;
        },
        todosPermisosSeleccionados() {
            return this.permisos.length > 0 &&
                this.permisosSeleccionados.length === this.permisos.length;
        },
        algunosPermisosSeleccionados() {
            return this.permisosSeleccionados.length > 0 &&
                this.permisosSeleccionados.length < this.permisos.length;
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
        validarGuardName(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.guard_name;

            if (!valor || valor.length < minLength) {
                this.errores.guard_name = `El Guard Name debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.guard_name = `El Guard Name no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.guard_name = 'El Guard Name contiene caracteres no válidos';
                return false;
            }

            this.errores.guard_name = '';
            return true;
        },
        validarEstado(valor) {
            if (!valor) {
                this.errores.estado = 'El estado es obligatorio';
                return false;
            }

            this.errores.estado = '';
            return true;
        },
        validarPermisos(permisos) {
            if (!permisos || permisos.length === 0) {
                this.errores.permissions = 'Debe seleccionar al menos un permiso';
                return false;
            }
            this.errores.permissions = '';
            return true;
        },
        async obtenerPermisos() {
            try {
                const response = await axios.get('/api/permissions'); // Asegúrate de tener una API para obtener permisos
                this.permisos = response.data;
            } catch (error) {
                console.error('Error al obtener permisos:', error);
            }
        },
        togglePermiso(permisoId) {
            const index = this.permisosSeleccionados.indexOf(permisoId);
            if (index === -1) {
                this.permisosSeleccionados.push(permisoId);
            } else {
                this.permisosSeleccionados.splice(index, 1);
            }
            // Validar después de cada cambio
            this.validarPermisos(this.permisosSeleccionados);
        },
        toggleTodosPermisos() {
            if (this.todosPermisosSeleccionados) {
                this.permisosSeleccionados = [];
            } else {
                this.permisosSeleccionados = this.permisos.map(p => p.id);
            }
            this.validarPermisos(this.permisosSeleccionados);
        },

        moduloCompleto(modulo) {
            const permisosDelModulo = this.permisosPorModulo[modulo];
            return permisosDelModulo.every(p => this.permisosSeleccionados.includes(p.id));
        },

        moduloParcial(modulo) {
            const permisosDelModulo = this.permisosPorModulo[modulo];
            const seleccionadosDelModulo = permisosDelModulo.filter(p =>
                this.permisosSeleccionados.includes(p.id)
            );
            return seleccionadosDelModulo.length > 0 &&
                seleccionadosDelModulo.length < permisosDelModulo.length;
        },

        toggleModulo(modulo) {
            const permisosDelModulo = this.permisosPorModulo[modulo];
            const todosSeleccionados = this.moduloCompleto(modulo);

            if (todosSeleccionados) {
                // Deseleccionar todos los permisos del módulo
                this.permisosSeleccionados = this.permisosSeleccionados.filter(id =>
                    !permisosDelModulo.some(p => p.id === id)
                );
            } else {
                // Seleccionar todos los permisos del módulo
                const nuevosPermisos = permisosDelModulo
                    .map(p => p.id)
                    .filter(id => !this.permisosSeleccionados.includes(id));
                this.permisosSeleccionados.push(...nuevosPermisos);
            }
            this.validarPermisos(this.permisosSeleccionados);
        },

        obtenerNombrePermiso(nombreCompleto) {
            // Extraer solo la acción del nombre del permiso (después del punto)
            const partes = nombreCompleto.split('.');
            return partes[partes.length - 1];
        },
        // Agregar los métodos faltantes
        formatearNombreModulo(modulo) {
            // Capitalizar primera letra y reemplazar guiones/puntos por espacios
            return modulo.charAt(0).toUpperCase() +
                modulo.slice(1).replace(/[-_.]/g, ' ');
        },

        formatearNombrePermiso(nombreCompleto) {
            // Extraer solo la acción del nombre del permiso (después del punto)
            const partes = nombreCompleto.split('.');
            const accion = partes[partes.length - 1];
            // Capitalizar primera letra y reemplazar guiones por espacios
            return accion.charAt(0).toUpperCase() +
                accion.slice(1).replace(/-/g, ' ');
        },
        toggleEstado() {
            this.rol.estado = this.rol.estado === 'Activo' ? 'Desactivado' : 'Activo';
        },
        async guardarRol() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones

                if (!this.validarName(this.rol.name)) {
                    return;
                }
                if (!this.validarGuardName(this.rol.guard_name)) {
                    return;
                }
                if (!this.validarEstado(this.rol.estado)) {
                    return;
                }
                const rolAEnviar = {
                    name: this.rol.name,
                    guard_name: this.rol.guard_name,
                    estado: this.rol.estado,
                    permissions: this.permisosSeleccionados // Incluir los permisos seleccionados
                };


                if (this.rolEditado) {
                    await axios.put(`/api/roles/${this.rolEditado.id}`, rolAEnviar);
                } else {
                    await axios.post('/api/roles', rolAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar el rol';
            }
        },
        cargarDatosEdicion() {
            if (this.rolEditado) {
                this.rol = {
                    name: this.rolEditado.name,
                    guard_name: this.rolEditado.guard_name,
                    estado: this.rolEditado.estado
                };
                // Cargar los permisos usando los IDs
                this.permisosSeleccionados = this.rolEditado.permissions.map(p => p.id);
            }
        }
    },
    created() {
        Promise.all([
            this.obtenerPermisos()
        ]).then(() => {
            if (this.rolEditado) {
                this.cargarDatosEdicion();
            }
        }).catch(error => {
            console.error("Error al cargar datos iniciales:", error);
            this.error = "Error al cargar los datos iniciales";
        });
    }
};
</script>

<template>
    <div class="container mt-0">
        <h2 class="mb-4 text-center">{{ rolEditado ? 'Editar' : 'Registrar' }} Rol</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>

        <form @submit.prevent="guardarRol" class="form-container">
            <div class="form-group mb-3">
                <label for="name">Nombre</label>
                <input v-model="rol.name" type="text" id="name" class="form-control"
                    :class="{ 'is-invalid': errores.name }" required>
                <div class="invalid-feedback" v-if="errores.name">
                    {{ errores.name }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="guard_name">Guard Name</label>
                <input v-model="rol.guard_name" type="text" id="guard_name" class="form-control"
                    :class="{ 'is-invalid': errores.guard_nameame }" required>
                <div class="invalid-feedback" v-if="errores.guard_name">
                    {{ errores.guard_name }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="estadoToggle">Estado</label>
                <div class="form-check form-switch">
                    <input type="checkbox" class="form-check-input" id="estadoToggle" :checked="rol.estado === 'Activo'"
                        @change="toggleEstado">
                    <label class="form-check-label" for="estadoToggle">{{ rol.estado }}</label>
                </div>
            </div>
            <div class="form-group mb-3">
                <label class="form-label fw-bold mb-3">Permisos</label>

                <!-- Checkbox principal para seleccionar todos los permisos -->
                <div class="form-check mb-3">
                    <input type="checkbox" class="form-check-input" id="seleccionar-todos"
                        :checked="todosPermisosSeleccionados" @change="toggleTodosPermisos">
                    <label class="form-check-label" for="seleccionar-todos">
                        <strong>Seleccionar todos los permisos</strong>
                    </label>
                </div>

                <!-- Lista de módulos y sus permisos -->
                <div class="permisos-container">
                    <div v-for="(permisos, modulo) in permisosPorModulo" :key="modulo" class="modulo-container">
                        <!-- Cabecera del módulo -->
                        <div class="modulo-header">
                            <input type="checkbox" class="form-check-input" :id="'modulo-' + modulo"
                                :checked="moduloCompleto(modulo)" :indeterminate.prop="moduloParcial(modulo)"
                                @change="toggleModulo(modulo)">
                            <label :for="'modulo-' + modulo" class="ms-2 text-primary">
                                {{ formatearNombreModulo(modulo) }}
                            </label>
                        </div>

                        <!-- Permisos del módulo -->
                        <div class="permisos-list">
                            <div v-for="permiso in permisos" :key="permiso.id" class="permiso-item">
                                <input type="checkbox" class="form-check-input" :id="'permiso-' + permiso.id"
                                    :value="permiso.id" :checked="permisosSeleccionados.includes(permiso.id)"
                                    @change="togglePermiso(permiso.id)">
                                <label :for="'permiso-' + permiso.id" class="ms-2">
                                    {{ formatearNombrePermiso(permiso.name) }}
                                </label>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="invalid-feedback" v-if="errores.permissions">
                    {{ errores.permissions }}
                </div>
            </div>
            <br>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ rolEditado ? 'Actualizar' : 'Registrar' }} Rol
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

.permisos-container {
    max-height: 400px;
    overflow-y: auto;
    border: 1px solid #dee2e6;
    border-radius: 0.25rem;
    padding: 1rem;
}

.modulo-container {
    margin-bottom: 1rem;
}

.modulo-header {
    font-weight: bold;
    margin-bottom: 0.5rem;
}

.permisos-list {
    margin-left: 1.5rem;
}

.permiso-item {
    margin-bottom: 0.25rem;
}
</style>
