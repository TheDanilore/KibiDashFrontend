<script>
import axios from 'axios';

export default {
    props: {
        salidaEditada: {
            type: Object,
            default: null
        },
    },
    data() {
        return {
            error: null,
            errores: {
                guia_salida: '',
                destino: '',
                productos: [] // Inicializamos como array vacío
            },
            productos: [],
            tiposSalida: [],
            salida: {
                guia_salida: '',
                tipo_salida: '',
                fecha: '',
                observaciones: '',
                destino: '',
                productos: []
            },
            productoSeleccionado: {
                producto_id: null,
                inventario_id: null, // Nuevo campo para el ID de inventario
                categoria: '',
                color_id: null,
                tamano_id: null,
                longitud_id: null
            },
            muestraColor: false,
            muestraTamaño: false,
            muestraLongitud: false,
            colores: [],
            tamanos: [],
            longitudes: [],
            validaciones: {
                guia_salida: {
                    minLength: 4,
                    maxLength: 20,
                    pattern: /^[A-Za-z0-9-]+$/
                },
                destino: {
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
                this.validarGuiaSalida(this.salida.guia_salida) &&
                this.salida.tipo_salida &&
                this.validarDestino(this.salida.destino) &&
                this.validarFecha(this.salida.fecha);

            // Validar que haya al menos un producto
            if (this.salida.productos.length === 0) {
                return false;
            }

            // Validar cada producto
            const productosValidos = this.salida.productos.every(p => {
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
        // Métodos de validación
        validarGuiaSalida(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.guia_salida;

            if (!valor || valor.length < minLength) {
                this.errores.guia_salida = `La guía de salida debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.guia_salida = `La guía de salida no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.guia_salida = 'La guía de salida solo puede contener letras, números y guiones';
                return false;
            }

            this.errores.guia_salida = '';
            return true;
        },

        validarDestino(valor) {
            const { minLength, maxLength, pattern } = this.validaciones.destino;

            if (!valor || valor.length < minLength) {
                this.errores.destino = `El destino debe tener al menos ${minLength} caracteres`;
                return false;
            }
            if (valor.length > maxLength) {
                this.errores.destino = `El destino no puede exceder ${maxLength} caracteres`;
                return false;
            }
            if (!pattern.test(valor)) {
                this.errores.destino = 'El destino contiene caracteres no válidos';
                return false;
            }

            this.errores.destino = '';
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
        async obtenerProductos() {
            try {
                const response = await axios.get('/api/productos');
                this.productos = response.data.data;
            } catch (error) {
                console.error('Error al obtener los productos:', error);
                this.error = 'Error al cargar los productos';
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
        async obtenerTiposSalida() {
            try {
                const response = await axios.get('/api/tipossalidas');
                this.tiposSalida = response.data;
            } catch (error) {
                console.error('Error al obtener tipos de salida:', error);
                this.error = 'Error al cargar los tipos de salida';
            }
        },
        async buscarInventario(producto_id, color_id, tamano_id, longitud_id) {
            try {
                // Construir los parámetros de manera dinámica
                const params = { producto_id };
                if (color_id !== null && color_id !== undefined) params.color_id = color_id;
                if (tamano_id !== null && tamano_id !== undefined) params.tamano_id = tamano_id;
                if (longitud_id !== null && longitud_id !== undefined) params.longitud_id = longitud_id;

                // Log para debugging
                console.log('Buscando inventario con parámetros:', params);

                const response = await axios.get('/api/buscarinventario', {
                    params,
                    // Añadir headers si es necesario
                    headers: {
                        'Accept': 'application/json',
                        'Content-Type': 'application/json'
                    }
                });

                // Log para debugging
                console.log('Respuesta del servidor:', response.data);

                if (response.data) {
                    return response.data;
                } else {
                    throw new Error('No se encontró inventario');
                }
            } catch (error) {
                console.error('Error al buscar inventario:', error);

                // Manejar diferentes tipos de errores
                if (error.response) {
                    // El servidor respondió con un código de error
                    const message = error.response.data.error || 'Error en la respuesta del servidor';
                    this.error = `Error ${error.response.status}: ${message}`;
                } else if (error.request) {
                    // La petición fue hecha pero no se recibió respuesta
                    this.error = 'No se pudo conectar con el servidor';
                } else {
                    // Error en la configuración de la petición
                    this.error = error.message;
                }

                return null;
            }
        },
        async seleccionarProducto() {
            try {
                const producto = this.productos.find(p => p.id === this.productoSeleccionado.producto_id);

                if (producto) {
                    // Llama al método de la API para obtener variantes específicas del inventario
                    const response = await axios.get(`/api/inventario/producto/${this.productoSeleccionado.producto_id}/resumen`);

                    // Asigna los datos obtenidos a las listas de variantes correspondientes
                    this.colores = response.data.colores || [];
                    this.tamanos = response.data.tamanos || [];
                    this.longitudes = response.data.longitudes || [];

                    // Actualiza la visibilidad de cada variante según los datos recibidos
                    this.muestraColor = this.colores.length > 0;
                    this.muestraTamaño = this.tamanos.length > 0;
                    this.muestraLongitud = this.longitudes.length > 0;

                    // Limpiar los valores de las variantes que no aplican
                    if (!this.muestraColor) this.productoSeleccionado.color_id = null;
                    if (!this.muestraTamaño) this.productoSeleccionado.tamano_id = null;
                    if (!this.muestraLongitud) this.productoSeleccionado.longitud_id = null;

                    // Validar y agregar producto si no existe duplicado
                    if (!this.verificarDuplicadoActual()) {
                        await this.agregarProducto();
                    } else {
                        this.error = 'Esta combinación de variantes ya ha sido añadida.';
                    }
                }
            } catch (error) {
                console.error('Error al seleccionar el producto:', error);
                this.error = 'Error al cargar las variantes de inventario';
            }
        },
        verificarDuplicadoActual() {
            const { producto_id, color_id, tamano_id, longitud_id } = this.productoSeleccionado;
            return this.salida.productos.some(v =>
                v.producto_id === producto_id &&
                (!this.muestraColor || v.color_id === color_id) &&
                (!this.muestraTamaño || v.tamano_id === tamano_id) &&
                (!this.muestraLongitud || v.longitud_id === longitud_id)
            );
        },
        async agregarProducto() {
            if (!this.productoSeleccionado.producto_id) return;

            try {
                // Solo enviar los IDs que correspondan según la categoría
                const inventarioData = await this.buscarInventario(
                    this.productoSeleccionado.producto_id,
                    this.muestraColor ? this.productoSeleccionado.color_id : null,
                    this.muestraTamaño ? this.productoSeleccionado.tamano_id : null,
                    this.muestraLongitud ? this.productoSeleccionado.longitud_id : null
                );
                if (!inventarioData) {
                    return; // El error ya se manejó en buscarInventario
                }
                const nuevaVariante = {
                    producto_id: this.productoSeleccionado.producto_id,
                    inventario_id: inventarioData.id,
                    color_id: this.muestraColor ? this.productoSeleccionado.color_id : null,
                    tamano_id: this.muestraTamaño ? this.productoSeleccionado.tamano_id : null,
                    longitud_id: this.muestraLongitud ? this.productoSeleccionado.longitud_id : null,
                    cantidad: 1,
                    precio_unitario: 0.01,
                    stock_disponible: inventarioData.cantidad
                };

                const existeDuplicado = this.verificarDuplicadoActual();

                if (!existeDuplicado) {
                    this.salida.productos.push(nuevaVariante);
                } else {
                    this.error = 'Esta combinación de variantes ya ha sido añadida.';
                }
            } catch (error) {
                console.error('Error al agregar producto:', error);
                this.error = 'Error al agregar el producto: ' + error.message;
            }
        },
        resetFormulario() {
            this.salida = { producto_id: null, cantidad: 1, fecha: '', observaciones: '' };
        },
        eliminarProducto(index) {
            this.salida.productos.splice(index, 1);
        },
        async guardarSalida() {
            try {
                this.error = null;
                // Verifica que todos los productos tengan un inventario_id
                for (const producto of this.salida.productos) {
                    if (!producto.inventario_id) {
                        throw new Error(`El producto ${producto.producto_id} no tiene un inventario_id.`);
                    }
                }
                // Validar todos los campos antes de enviar
                if (!this.esFormularioValido) {
                    this.error = 'Por favor, corrija los errores antes de guardar';
                    return;
                }
                // Validar observaciones
                if (!this.validarObservaciones(this.salida.observaciones)) {
                    return;
                }
                const salidaAEnviar = {
                    ...this.salida,
                    productos: this.salida.productos.map(p => ({
                        producto_id: p.producto_id,
                        inventario_id: p.inventario_id,
                        cantidad: p.cantidad,
                        precio_unitario: p.precio_unitario,
                        color_id: p.color_id,
                        tamano_id: p.tamano_id,
                        longitud_id: p.longitud_id,
                    }))
                };

                await axios.post('/api/salidasproductos', salidaAEnviar);
                this.$emit('guardar');
            } catch (error) {
                console.error('Error completo:', error);
                this.error = error.response?.data?.error || 'Error al guardar la salida';
            }
        }

    },
    created() {
        if (this.salidaEditada) {
            this.salida = { ...this.salidaEditada };
            if (this.salida.productos && this.salida.productos.length > 0) {
                this.productoSeleccionado.producto_id = this.salida.productos[0].producto_id;
                this.seleccionarProducto();
            }
        }
        this.obtenerProductos();
        this.obtenerTiposSalida();
        this.obtenerColores();
        this.obtenerTamanos();
        this.obtenerLongitudes();
        this.obtenerCategorias();
    }
};
</script>


<template>
    <div class="container mt-0">
        <h2 class="mb-4 text-center">{{ salidaEditada ? 'Editar' : 'Registrar' }} Salida de Producto</h2>

        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>
        <form @submit.prevent="guardarSalida">
            <!-- Datos básicos -->
            <div class="form-group mb-3">
                <label for="guia_salida">Guía de Salida</label>
                <input v-model="salida.guia_salida" type="text" class="form-control"
                    :class="{ 'is-invalid': errores.guia_salida }" required>
                <div class="invalid-feedback" v-if="errores.guia_salida">
                    {{ errores.guia_salida }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="tipo_salida">Tipo de Salida</label>
                <select name="tipo_salida" v-model="salida.tipo_salida" class="form-control" required>
                    <option value="">Seleccione un tipo de salida</option>
                    <option v-for="tipo in tiposSalida" :key="tipo" :value="tipo">
                        {{ tipo }}
                    </option>
                </select>
            </div>
            <div class="form-group mb-3">
                <label for="destino">Destino</label>
                <input v-model="salida.destino" type="text" class="form-control"
                    :class="{ 'is-invalid': errores.destino }" required>
                <div class="invalid-feedback" v-if="errores.destino">
                    {{ errores.destino }}
                </div>
            </div>
            <div class="form-group mb-3">
                <label for="fecha">Fecha de Salida</label>
                <input v-model="salida.fecha" type="date" id="fecha" class="form-control" required
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
                        <div v-for="(variante, index) in salida.productos" :key="index" class="border p-3 mb-3">
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
                <textarea v-model="salida.observaciones" class="form-control" rows="3"
                    :maxlength="validaciones.observaciones.maxLength">
                </textarea>
                <small class="text-muted">
                    Caracteres restantes: {{ validaciones.observaciones.maxLength - (salida.observaciones?.length || 0)
                    }}
                </small>
            </div>

            <div class="d-flex justify-content-between">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ salidaEditada ? 'Actualizar' : 'Registrar' }} Salida
                </button>
            </div>
        </form>
    </div>
</template>
