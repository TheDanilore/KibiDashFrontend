<script>
import axios from 'axios';

export default {
    props: {
        categoriaEditada: {
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
            categoria: {
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
                this.validarDescripcion(this.categoria.descripcion);

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
        async guardarCategoria() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones
                if (!this.validarDescripcion(this.categoria.descripcion)) {
                    return;
                }
                const categoriaAEnviar = {
                    ...this.categoria
                };


                if (this.categoriaEditada) {
                    await axios.put(`/api/categoriaproductos/${this.categoriaEditada.id}`, categoriaAEnviar);
                } else {
                    await axios.post('/api/categoriaproductos', categoriaAEnviar);
                }
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar la categoria';
            }
        },
    },

    created() {
        Promise.all([
        ]).then(() => {
            if (this.categoriaEditada) {
                this.modo = 'editar';
                this.categoria = { ...this.categoriaEditada };
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
        <h2 class="mb-4 text-center">{{ categoriaEditada ? 'Editar' : 'Registrar' }} Categoría</h2>
        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>


        <form @submit.prevent="guardarCategoria" class="form-container">
            <div class="form-group mb-3">
                <label for="descripcion">Descripción</label>
                <input v-model="categoria.descripcion" type="text" id="descripcion" class="form-control"
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
                    {{ categoriaEditada ? 'Actualizar' : 'Registrar' }} Categoría
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

.btn {
    margin: 0 5px;
    /* Espacio entre los botones */
}

.mt-4 {
    margin-top: 1.5rem;
}
</style>
