<script>
import axios from 'axios';

export default {
    props: {
        ubicacionEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                descripcion: '',
                codigo: '',
            },
            ubicacion: {
                descripcion: '',
                codigo: '',
            },
            validaciones: {
                descripcion: {
                    minLength: 5,
                    maxLength: 20,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
                codigo: {
                    minLength: 4,
                    maxLength: 8,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
            }
        };
    },
    computed: {
        esFormularioValido() {
            // Validar campos básicos
            const camposBasicosValidos =
                this.validarDescripcion(this.ubicacion.descripcion) && this.validarCodigo(this.ubicacion.codigo);

            return camposBasicosValidos;
        }
    },
    methods: {
        validarDescripcion(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.descripcion;

            if (!valor || valor.length < minLength) {
                this.errores.descripcion = `La descripcion debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.descripcion = `La descripcion no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.descripcion = 'La descripcion contiene caracteres no válidos';
                return false;
            }

            this.errores.descripcion = '';
            return true;
        },
        validarCodigo(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.codigo;

            if (!valor || valor.length < minLength) {
                this.errores.codigo = `El codigo debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.codigo = `El codigo no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.codigo = 'El codigo contiene caracteres no válidos';
                return false;
            }

            this.errores.codigo = '';
            return true;
        },
        async guardarUbicacion() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones
                if (!this.validarDescripcion(this.ubicacion.descripcion)) {
                    return;
                }
                if (!this.validarCodigo(this.ubicacion.codigo)) {
                    return;
                }
                const ubicacionAEnviar = {
                    ...this.ubicacion
                };

                if (this.ubicacionEditado) {
                    await axios.put(`/api/ubicaciones/${this.ubicacionEditado.id}`, ubicacionAEnviar);
                } else {
                    await axios.post('/api/ubicaciones', ubicacionAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar el tamaño';
            }
        }
    },
    created() {
        Promise.all([
        ]).then(() => {
            if (this.ubicacionEditado) {
                this.modo = 'editar';
                this.ubicacion = { ...this.ubicacionEditado };
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
        <h2 class="mb-4 text-center">{{ ubicacionEditado ? 'Editar' : 'Registrar' }} Ubicación</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>

        <form @submit.prevent="guardarUbicacion" class="form-container">
            <div class="form-group mb-3">
                <label for="descripcion">Descripción</label>
                <input v-model="ubicacion.descripcion" type="text" id="descripcion" class="form-control"
                    :class="{ 'is-invalid': errores.descripcion }" required>
                <div class="invalid-feedback" v-if="errores.descripcion">
                    {{ errores.descripcion }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="codigo">Código</label>
                <input v-model="ubicacion.codigo" type="text" id="codigo" class="form-control"
                    :class="{ 'is-invalid': errores.codigo }" required>
                <div class="invalid-feedback" v-if="errores.codigo">
                    {{ errores.codigo }}
                </div>
            </div>
            <br>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ ubicacionEditado ? 'Actualizar' : 'Registrar' }} Ubicación
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
</style>
