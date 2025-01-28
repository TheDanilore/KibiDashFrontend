<script>
import axios from 'axios';

export default {
    props: {
        entradaEditada: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                guia_remision: '',
                procedencia: '',
                productos: [] // Inicializamos como array vacío
            },
            colores: [],
            tamanos: [],
            longitudes: [],
            productos: [],
            categorias: [],
            proveedores: [],
            tiposEntrada: [],
            entrada: {
                guia_remision: '',
                proveedor_id: '',
                tipo_entrada: '',
                procedencia: '',
                fecha: '',
                observaciones: '',
                productos: []
            },
            productoSeleccionado: {
                producto_id: '',
                categoria: '',
                color_id: '',
                tamano_id: '',
                longitud_id: ''
            },
            muestraColor: false,
            muestraTamaño: false,
            muestraLongitud: false,
            validaciones: {
                guia_remision: {
                    minLength: 4,
                    maxLength: 20,
                    pattern: /^[A-Za-z0-9-]+$/
                },
                procedencia: {
                    minLength: 4,
                    maxLength: 60,
                    pattern: /^[A-Za-z0-9\s.,#-]+$/
                },
                cantidad: {
                    min: 1,
                    max: 999999
                },
                precio_unitario: {
                    min: 0.01,
                    max: 99999.99,
                    decimales: 2
                },
                observaciones: {
                    maxLength: 190
                }
            }
        };
    },
    computed: {
        esFormularioValido() {
            // Validar campos básicos
            const camposBasicosValidos =
                this.validarGuiaRemision(this.entrada.guia_remision) &&
                this.entrada.proveedor_id &&
                this.entrada.tipo_entrada &&
                this.validarProcedencia(this.entrada.procedencia) &&
                this.validarFecha(this.entrada.fecha);

            // Validar que haya al menos un producto
            if (this.entrada.productos.length === 0) {
                return false;
            }

            // Validar cada producto
            const productosValidos = this.entrada.productos.every(p => {
                const tieneVariantesRequeridas =
                    (!this.muestraColor || p.color_id) &&
                    (!this.muestraTamaño || p.tamano_id) &&
                    (!this.muestraLongitud || p.longitud_id);

                // Validamos cada campo independientemente
                const cantidadValida = this.validarCantidad(p.cantidad);
                const precioValido = this.validarPrecioUnitario(p.precio_unitario);

                return p.producto_id &&
                    tieneVariantesRequeridas &&
                    cantidadValida &&
                    precioValido;
            });

            return camposBasicosValidos && productosValidos;
        }
    },
    methods: {
        async obtenerProductos() {
            try {
                const response = await axios.get('/api/productos');
                this.productos = response.data.data;
            } catch (error) {
                console.error('Error al obtener los productos:', error);
                this.error = 'Error al cargar los productos';
            }
        },
        async obtenerProveedores() {
            try {
                const response = await axios.get('/api/proveedores');
                this.proveedores = response.data;
            } catch (error) {
                console.error('Error al obtener los proveedores:', error);
                this.error = 'Error al cargar los proveedores';
            }
        },
        async obtenerCategorias() {
            try {
                const response = await axios.get('/api/categoriaproductos'); // Asegúrate de que esta URL sea correcta
                this.categorias = response.data;
            } catch (error) {
                console.error('Error al obtener las categorías:', error);
                this.error = 'Error al cargar las categorías';
            }
        },
        async obtenerColores() {
            try {
                const response = await axios.get('/api/colores');
                this.colores = response.data;
            } catch (error) {
                console.error('Error al obtener colores:', error);
                this.error = 'Error al cargar los colores';
            }
        },
        async obtenerTamanos() {
            try {
                const response = await axios.get('/api/tamanos');
                this.tamanos = response.data;
            } catch (error) {
                console.error('Error al obtener tamaños:', error);
                this.error = 'Error al cargar los tamaños';
            }
        },
        async obtenerLongitudes() {
            try {
                const response = await axios.get('/api/longitudes');
                this.longitudes = response.data;
            } catch (error) {
                console.error('Error al obtener longitudes:', error);
                this.error = 'Error al cargar las longitudes';
            }
        },
        async obtenerTiposEntrada() {
            try {
                const response = await axios.get('/api/tiposentradas');
                this.tiposEntrada = response.data;
            } catch (error) {
                console.error('Error al obtener tipos de entrada:', error);
                this.error = 'Error al cargar los tipos de entrada';
            }
        },
        async seleccionarProducto() {
            const producto = this.productos.find(p => p.id === this.productoSeleccionado.producto_id);
            if (producto) {
                if (!this.categorias.length) {
                    await this.obtenerCategorias(); // Asegura la carga de categorías
                }

                const categoria = this.categorias.find(c => String(c.id) === String(producto.categoria_producto_id));
                if (categoria) {
                    this.muestraColor = ['Ropa', 'Articulos', 'Polos', 'Collares'].includes(categoria.descripcion);
                    this.muestraTamaño = ['Ropa', 'Polos'].includes(categoria.descripcion);
                    this.muestraLongitud = ['Articulos', 'Collares'].includes(categoria.descripcion);

                    // Carga las variantes de inventario del producto seleccionado
                    try {
                        const response = await axios.get(`/api/inventario/producto/${this.productoSeleccionado.producto_id}/resumen`);
                        const variantes = response.data;

                        this.colores = variantes.colores || [];
                        this.tamanos = variantes.tamanos || [];
                        this.longitudes = variantes.longitudes || [];
                    } catch (error) {
                        console.error('Error al obtener variantes del inventario:', error);
                        this.error = 'Error al cargar las variantes de producto';
                    }

                    if (!this.verificarDuplicadoActual()) {
                        this.agregarProducto();
                    } else {
                        this.error = 'Esta combinación de variantes ya ha sido añadida.';
                    }
                } else {
                    console.error('Categoría no encontrada para el producto seleccionado');
                }
            }
        },
        verificarDuplicadoActual() {
            const { producto_id, color_id, tamano_id, longitud_id } = this.productoSeleccionado;
            return this.entrada.productos.some(v =>
                v.producto_id === producto_id &&
                (!this.muestraColor || v.color_id === color_id) &&
                (!this.muestraTamaño || v.tamano_id === tamano_id) &&
                (!this.muestraLongitud || v.longitud_id === longitud_id)
            );
        },
        // Métodos de validación
        validarGuiaRemision(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.guia_remision;

            if (!valor || valor.length < minLength) {
                this.errores.guia_remision = `La guía de remisión debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.guia_remision = `La guía de remisión no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.guia_remision = 'La guía de remisión solo puede contener letras, números y guiones';
                return false;
            }

            this.errores.guia_remision = '';
            return true;
        },

        validarProcedencia(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.procedencia;

            if (!valor || valor.length < minLength) {
                this.errores.procedencia = `La procedencia debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.procedencia = `La procedencia no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.procedencia = 'La procedencia contiene caracteres no válidos';
                return false;
            }

            this.errores.procedencia = '';
            return true;
        },

        validarFecha(valor) {
            if (!valor) return false;

            const fecha = new Date(valor);
            const hoy = new Date();

            // No permitir fechas futuras
            if (fecha > hoy) {
                this.error = 'La fecha no puede ser futura';
                return false;
            }

            // No permitir fechas anteriores a 1 año
            const unAnoAtras = new Date();
            unAnoAtras.setFullYear(hoy.getFullYear() - 1);
            if (fecha < unAnoAtras) {
                this.error = 'La fecha no puede ser anterior a un año';
                return false;
            }

            return true;
        },
        // Actualizar el método validarCantidad para que maneje los errores por índice
        validarCantidad(valor, index) {
            const { min, max } = this.validaciones.cantidad;

            // Asegurarse de que el array de errores para productos existe
            if (!this.errores.productos[index]) {
                this.errores.productos[index] = {};
            }

            if (!Number.isInteger(valor) || valor < min || valor > max) {
                this.errores.productos[index].cantidad =
                    `La cantidad debe ser un número entero entre ${min} y ${max}`;
                return false;
            }

            this.errores.productos[index].cantidad = '';
            return true;
        },
        // Actualizar el método validarPrecioUnitario para que maneje los errores por índice
        validarPrecioUnitario(valor, index) {
            const { min, max, decimales } = this.validaciones.precio_unitario;

            // Asegurarse de que el array de errores para productos existe
            if (!this.errores.productos[index]) {
                this.errores.productos[index] = {};
            }

            if (isNaN(valor) || valor < min || valor > max) {
                this.errores.productos[index].precio_unitario =
                    `El precio debe estar entre ${min} y ${max}`;
                return false;
            }

            const partes = valor.toString().split('.');
            if (partes.length > 1 && partes[1].length > decimales) {
                this.errores.productos[index].precio_unitario =
                    `El precio no puede tener más de ${decimales} decimales`;
                return false;
            }

            this.errores.productos[index].precio_unitario = '';
            return true;
        },
        validarObservaciones(valor) {
            if (valor && valor.length > this.validaciones.observaciones.maxLength) {
                this.error = `Las observaciones no pueden exceder ${this.validaciones.observaciones.maxLength} caracteres`;
                return false;
            }
            return true;
        },
        // Agregar un nuevo producto a la lista de productos
        agregarProducto() {
            if (!this.productoSeleccionado.producto_id) return;

            const nuevaVariante = {
                producto_id: this.productoSeleccionado.producto_id,
                color_id: this.muestraColor ? this.productoSeleccionado.color_id : '',
                tamano_id: this.muestraTamaño ? this.productoSeleccionado.tamano_id : '',
                longitud_id: this.muestraLongitud ? this.productoSeleccionado.longitud_id : '',
                cantidad: 1,
                precio_unitario: 0.01
            };

            const existeDuplicado = this.verificarDuplicadoActual();
            if (!existeDuplicado) {
                // Inicializar el objeto de errores para el nuevo producto
                this.errores.productos.push({
                    cantidad: '',
                    precio_unitario: ''
                });

                // Agregar el nuevo producto
                this.entrada.productos.push(nuevaVariante);
                this.error = null;
            } else {
                this.error = 'Esta combinación de variantes ya ha sido añadida.';
            }
        },
        eliminarProducto(index) {
            this.entrada.productos.splice(index, 1);
        },
        async guardarEntrada() {
            try {
                this.error = null;
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones
                if (!this.validarObservaciones(this.entrada.observaciones)) {
                    return;
                }
                const entradaAEnviar = {
                    ...this.entrada,
                    productos: this.entrada.productos.map(p => ({
                        ...p,
                        producto_id: this.productoSeleccionado.producto_id
                    }))
                };

                if (this.entradaEditada) {
                    await axios.put(`/api/entradasproductos/${this.entradaEditada.id}`, entradaAEnviar);
                } else {
                    await axios.post('/api/entradasproductos', entradaAEnviar);
                }

                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error ||
                    error.response?.data?.message ||
                    'Error al guardar la entrada';
            }
        },
    },
    created() {
        Promise.all([
            this.obtenerProductos(),
            this.obtenerProveedores(),
            this.obtenerTiposEntrada(),
            this.obtenerColores(),
            this.obtenerTamanos(),
            this.obtenerLongitudes(),
            this.obtenerCategorias()
        ]).then(() => {
            if (this.entradaEditada) {
                this.entrada = { ...this.entradaEditada };
                if (this.entrada.productos && this.entrada.productos.length > 0) {
                    this.productoSeleccionado.producto_id = this.entrada.productos[0].producto_id;
                    this.seleccionarProducto();
                }
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
        <h2 class="mb-4 text-center">{{ entradaEditada ? 'Editar' : 'Registrar' }} Entrada de Producto</h2>

        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>

        <form @submit.prevent="guardarEntrada">
            <!-- Datos básicos -->
            <div class="form-group mb-3">
                <label for="guia_remision">Guía de Remisión</label>
                <input v-model="entrada.guia_remision" type="text" class="form-control"
                    :class="{ 'is-invalid': errores.guia_remision }" required>
                <div class="invalid-feedback" v-if="errores.guia_remision">
                    {{ errores.guia_remision }}
                </div>
            </div>

            <div class="form-group mb-3">
                <label for="proveedor">Proveedor</label>
                <select v-model="entrada.proveedor_id" class="form-control" required>
                    <option value="">Seleccione un proveedor</option>
                    <option v-for="proveedor in proveedores" :key="proveedor.id" :value="proveedor.id">
                        {{ proveedor.razon_social }}
                    </option>
                </select>
            </div>

            <div class="form-group mb-3">
                <label for="tipo_entrada">Tipo de Entrada</label>
                <select name="tipo_entrada" v-model="entrada.tipo_entrada" class="form-control" required>
                    <option value="">Seleccione un tipo de entrada</option>
                    <option v-for="tipo in tiposEntrada" :key="tipo" :value="tipo">
                        {{ tipo }}
                    </option>
                </select>
            </div>

            <div class="form-group mb-3">
                <label for="procedencia">Procedencia</label>
                <input v-model="entrada.procedencia" type="text" class="form-control"
                    :class="{ 'is-invalid': errores.procedencia }" required>
                <div class="invalid-feedback" v-if="errores.procedencia">
                    {{ errores.procedencia }}
                </div>
            </div>

            <div class="form-group mb-3">
                <label for="fecha">Fecha de Entrada</label>
                <input v-model="entrada.fecha" type="date" id="fecha" class="form-control" required
                    :class="{ 'is-invalid': errores.fecha }">
                <div class="invalid-feedback" v-if="errores.fecha">
                    {{ errores.fecha }}
                </div>
            </div>

            <!-- Sección de Productos -->
            <div class="card mb-3">
                <div class="card-header">
                    <h5 class="mb-0">Información del Producto</h5>
                </div>
                <div class="card-body">
                    <!-- Selección del producto principal -->
                    <div class="form-group mb-3">
                        <label>Seleccionar Producto</label>
                        <select v-model="productoSeleccionado.producto_id" class="form-control"
                            @change="seleccionarProducto" required>
                            <option value="">Seleccione un producto</option>
                            <option v-for="producto in productos" :key="producto.id" :value="producto.id">
                                {{ producto.nombre }}
                            </option>
                        </select>
                    </div>

                    <!-- Variantes y detalles del producto -->
                    <div v-if="productoSeleccionado.producto_id">
                        <div v-for="(variante, index) in entrada.productos" :key="index" class="border p-3 mb-3">
                            <div class="d-flex justify-content-end mb-2">
                                <button type="button" class="btn btn-danger btn-sm" @click="eliminarProducto(index)">
                                    Eliminar Variante
                                </button>
                            </div>

                            <!-- Campos de variantes según la categoría -->
                            <div class="row">
                                <div v-if="muestraColor" class="col-md-4">
                                    <div class="form-group mb-3">
                                        <label>Color</label>
                                        <select v-model="variante.color_id" class="form-control" required>
                                            <option value="">Seleccione un color</option>
                                            <option v-for="color in colores" :key="color.id" :value="color.id">
                                                {{ color.descripcion }}
                                            </option>
                                        </select>
                                    </div>
                                </div>

                                <div v-if="muestraTamaño" class="col-md-4">
                                    <div class="form-group mb-3">
                                        <label>Tamaño</label>
                                        <select v-model="variante.tamano_id" class="form-control" required>
                                            <option value="">Seleccione un tamaño</option>
                                            <option v-for="tamano in tamanos" :key="tamano.id" :value="tamano.id">
                                                {{ tamano.descripcion }}
                                            </option>
                                        </select>
                                    </div>
                                </div>

                                <div v-if="muestraLongitud" class="col-md-4">
                                    <div class="form-group mb-3">
                                        <label>Longitud</label>
                                        <select v-model="variante.longitud_id" class="form-control" required>
                                            <option value="">Seleccione una longitud</option>
                                            <option v-for="longitud in longitudes" :key="longitud.id"
                                                :value="longitud.id">
                                                {{ longitud.descripcion }}
                                            </option>
                                        </select>
                                    </div>
                                </div>
                            </div>

                            <!-- Campos de cantidad y precio -->
                            <div class="row">
                                <div class="col-md-6">
                                    <div class="form-group mb-3">
                                        <label>Cantidad</label>
                                        <input type="number" v-model.number="variante.cantidad" class="form-control"
                                            :class="{ 'is-invalid': errores.productos[index]?.cantidad }"
                                            @input="validarCantidad(variante.cantidad, index)" required min="1"
                                            :max="validaciones.cantidad.max">
                                        <div class="invalid-feedback" v-if="errores.productos[index]?.cantidad">
                                            {{ errores.productos[index].cantidad }}
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-6">
                                    <div class="form-group mb-3">
                                        <label>Precio Unitario</label>
                                        <input type="number" v-model.number="variante.precio_unitario"
                                            class="form-control"
                                            :class="{ 'is-invalid': errores.productos[index]?.precio_unitario }"
                                            @input="validarPrecioUnitario(variante.precio_unitario, index)" required
                                            :min="validaciones.precio_unitario.min"
                                            :max="validaciones.precio_unitario.max"
                                            :step="Math.pow(10, -validaciones.precio_unitario.decimales)">
                                        <div class="invalid-feedback" v-if="errores.productos[index]?.precio_unitario">
                                            {{ errores.productos[index].precio_unitario }}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Botón para añadir más variantes -->
                        <div class="text-center mt-3">
                            <button type="button" class="btn btn-primary" @click="agregarProducto">
                                Añadir Otra Variante
                            </button>
                        </div>
                    </div>
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="observaciones">Observaciones</label>
                <textarea v-model="entrada.observaciones" class="form-control" rows="3"
                    :maxlength="validaciones.observaciones.maxLength">
                </textarea>
                <small class="text-muted">
                    Caracteres restantes: {{ validaciones.observaciones.maxLength - (entrada.observaciones?.length || 0)
                    }}
                </small>
            </div>

            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ entradaEditada ? 'Actualizar' : 'Registrar' }} Entrada
                </button>
            </div>
        </form>
    </div>
</template>