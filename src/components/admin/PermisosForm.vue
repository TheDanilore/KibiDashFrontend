<script>
import axios from 'axios';

export default {
    props: {
        permisoEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                name: '',
                guard_name: ''
            },
            permiso: {
                name: '',
                guard_name: 'web',
            },
            validaciones: {
                name: {
                    minLength: 4,
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
            // Validar campos básicos
            const camposBasicosValidos =
                this.validarName(this.permiso.name) &&
                this.validarGuardName(this.permiso.guard_name);

            return camposBasicosValidos;
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
        async guardarPermiso() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones

                if (!this.validarName(this.permiso.name)) {
                    return;
                }
                if (!this.validarGuardName(this.permiso.guard_name)) {
                    return;
                }
                const permisoAEnviar = {
                    ...this.permiso
                };


                if (this.permisoEditado) {
                    await axios.put(`/api/permissions/${this.permisoEditado.id}`, permisoAEnviar);
                } else {
                    await axios.post('/api/permissions', permisoAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar el permiso';
            }
        },
    },
    created() {
        Promise.all([
        ]).then(() => {
            if (this.permisoEditado) {
                this.modo = 'editar';
                this.permiso = { ...this.permisoEditado };
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
        <h2 class="mb-4 text-center">{{ permisoEditado ? 'Editar' : 'Registrar' }} Permiso</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>

        <form @submit.prevent="guardarPermiso" class="form-container">
            <div class="form-group mb-3">
                <label for="name">Nombre</label>
                <input v-model="permiso.name" type="text" id="name" class="form-control"
                    :class="{ 'is-invalid': errores.name }" required>
                <div class="invalid-feedback" v-if="errores.name">
                    {{ errores.name }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="guard_name">Guard Name</label>
                <input v-model="permiso.guard_name" type="text" id="guard_name" class="form-control"
                    :class="{ 'is-invalid': errores.guard_nameame }" required>
                <div class="invalid-feedback" v-if="errores.guard_name">
                    {{ errores.guard_name }}
                </div>
            </div>
            <br>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ permisoEditado ? 'Actualizar' : 'Registrar' }} Permiso
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
</style>
