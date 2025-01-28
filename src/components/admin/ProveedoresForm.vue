<script>
import axios from 'axios';

export default {
    props: {
        proveedorEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                ruc: '',
                razon_social: '',
                direccion: '',
                telefono: '',
                estado: ''
            },
            proveedor: {
                ruc: '',
                razon_social: '',
                direccion: '',
                telefono: '',
                estado: 'Activo'
            },
            validaciones: {
                ruc: {
                    minLength: 11,
                    maxLength: 11,
                    pattern: /^[0-9]+$/
                },
                razon_social: {
                    minLength: 3,
                    maxLength: 40,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
                direccion: {
                    minLength: 3,
                    maxLength: 70,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
                telefono: {
                    minLength: 9,
                    maxLength: 12,
                    pattern: /^[0-9-+]+$/
                },
            }
        };
    },
    computed: {
        esFormularioValido() {
            // Validar campos básicos
            const camposBasicosValidos =
                this.validarRuc(this.proveedor.ruc) &&
                this.validarRazonSocial(this.proveedor.razon_social) &&
                this.validarDireccion(this.proveedor.direccion) &&
                this.validarTelefono(this.proveedor.telefono) &&
                this.validarEstado(this.proveedor.estado);

            return camposBasicosValidos;
        }
    },
    methods: {
        validarRuc(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.ruc;

            if (!valor || valor.length < minLength) {
                this.errores.ruc = `El RUC debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.ruc = `El RUC no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.ruc = 'El RUC solo puede contener números';
                return false;
            }

            this.errores.ruc = '';
            return true;
        },
        validarRazonSocial(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.razon_social;

            if (!valor || valor.length < minLength) {
                this.errores.razon_social = `La razón social debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.razon_social = `La razón social no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.razon_social = 'La razón social solo puede contener letras, números y guiones';
                return false;
            }

            this.errores.razon_social = '';
            return true;
        },
        validarDireccion(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.direccion;

            if (!valor || valor.length < minLength) {
                this.errores.direccion = `La dirección debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.direccion = `La dirección no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.direccion = 'La dirección solo puede contener letras, números y guiones';
                return false;
            }

            this.errores.direccion = '';
            return true;
        },
        validarTelefono(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.telefono;

            if (!valor || valor.length < minLength) {
                this.errores.telefono = `El teléfono debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.telefono = `El teléfono no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.telefono = 'El teléfono solo puede contener números';
                return false;
            }

            this.errores.telefono = '';
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
        toggleEstado() {
            // Cambia directamente el estado basado en el valor actual
            this.proveedor.estado = this.proveedor.estado === 'Activo' ? 'Inactivo' : 'Activo';
            console.log('Nuevo estado:', this.proveedor.estado); // Para debugging
        },
        async guardarProveedor() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones

                if (!this.validarRuc(this.proveedor.ruc)) {
                    return;
                }
                if (!this.validarRazonSocial(this.proveedor.razon_social)) {
                    return;
                }
                if (!this.validarDireccion(this.proveedor.direccion)) {
                    return;
                }
                if (!this.validarTelefono(this.proveedor.telefono)) {
                    return;
                }
                if (!this.validarEstado(this.proveedor.estado)) {
                    return;
                }

                const proveedorAEnviar = {
                    ...this.proveedor
                };

                if (this.proveedorEditado) {
                    await axios.put(`/api/proveedores/${this.proveedorEditado.id}`, proveedorAEnviar);
                } else {
                    await axios.post('/api/proveedores', proveedorAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar el proveedor';
            }
        },
    },
    created() {
        Promise.all([
        ]).then(() => {
            if (this.proveedorEditado) {
                this.modo = 'editar';
                this.proveedor = { ...this.proveedorEditado };
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
        <h2 class="mb-4 text-center">{{ proveedorEditado ? 'Editar' : 'Registrar' }} Proveedor</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>
        <form @submit.prevent="guardarProveedor">
            <div class="form-group mb-3">
                <label for="ruc">RUC</label>
                <input v-model="proveedor.ruc" type="text" id="ruc" class="form-control"
                    :class="{ 'is-invalid': errores.ruc }" required>
                <div class="invalid-feedback" v-if="errores.ruc">
                    {{ errores.ruc }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="razon_social">Razon Social</label>
                <input v-model="proveedor.razon_social" type="text" id="razon_social" class="form-control"
                    :class="{ 'is-invalid': errores.razon_social }" required>
                <div class="invalid-feedback" v-if="errores.razon_social">
                    {{ errores.razon_social }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="direccion">Dirección</label>
                <input v-model="proveedor.direccion" type="text" id="direccion" class="form-control"
                    :class="{ 'is-invalid': errores.direccion }" required>
                <div class="invalid-feedback" v-if="errores.direccion">
                    {{ errores.direccion }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="telefono">Telélefono</label>
                <input v-model="proveedor.telefono" type="text" id="telefono" class="form-control"
                    :class="{ 'is-invalid': errores.telefono }" required>
                <div class="invalid-feedback" v-if="errores.telefono">
                    {{ errores.telefono }}
                </div>
            </div>
            <div class="form-group">
                <label for="estadoToggle">Estado</label>
                <div class="form-check form-switch">
                    <input type="checkbox" class="form-check-input" id="estadoToggle"
                        :checked="proveedor.estado === 'Activo'" @change="toggleEstado">
                    <label class="form-check-label" for="estadoToggle">
                        {{ proveedor.estado }}
                    </label>
                </div>
            </div>
            <br>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ proveedorEditado ? 'Actualizar' : 'Registrar' }} Proveedor
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
