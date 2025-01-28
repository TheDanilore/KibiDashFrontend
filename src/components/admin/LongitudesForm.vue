<script>
import axios from 'axios';

export default {
    props: {
        longitudEditado: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                descripcion: '',
            },
            longitud: {
                descripcion: '',
            },
            validaciones: {
                descripcion: {
                    minLength: 4,
                    maxLength: 80,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
            }
        };
    },
    computed: {
        esFormularioValido() {
            // Validar campos básicos
            const camposBasicosValidos =
                this.validarDescripcion(this.longitud.descripcion);

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
        async guardarLongitud() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones
                if (!this.validarDescripcion(this.longitud.descripcion)) {
                    return;
                }
                const longitudAEnviar = {
                    ...this.longitud
                };


                if (this.longitudEditado) {
                    await axios.put(`/api/longitudes/${this.longitudEditado.id}`, longitudAEnviar);
                } else {
                    await axios.post('/api/longitudes', longitudAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar la longitud';
            }
        },
    },
    created() {
        Promise.all([
        ]).then(() => {
            if (this.longitudEditado) {
                this.modo = 'editar';
                this.longitud = { ...this.longitudEditado };
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
        <h2 class="mb-4 text-center">{{ longitudEditado ? 'Editar' : 'Registrar' }} Longitud</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>

        <form @submit.prevent="guardarLongitud" class="form-container">
            <div class="form-group mb-3">
                <label for="descripcion">Descripción</label>
                <input v-model="longitud.descripcion" type="text" id="descripcion" class="form-control"
                    :class="{ 'is-invalid': errores.descripcion }" required>
                <div class="invalid-feedback" v-if="errores.descripcion">
                    {{ errores.descripcion }}
                </div>
            </div>
            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ longitudEditado ? 'Actualizar' : 'Registrar' }} Longitud
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
