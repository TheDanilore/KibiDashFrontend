<script>
import axios from 'axios';

export default {
    props: {
        productoEditado: {
            type: Object,
            default: () => ({}),
        },
    },
    emits: ['guardar', 'cerrar', 'productos-actualizados'],
    data() {
        return {
            error: null,
            producto: {
                nombre: '',
                descripcion: '',
                categoria_producto_id: null,
                unidad_medida_id: null,
                ubicacion_id: null,
                proveedor_id: null,
                imagenes: [],
                estado: 'Activo',
                colores: [],
                tamanos: [],
                longitudes: [],
            },
            variantes: [{
                color_id: null,
                tamano_id: null,
                longitud_id: null,
                precio_unitario: 0,
                cantidad: 0
            }],
            // Añadimos las propiedades faltantes al data()
            categorias: [],        // Array para las categorías
            colores: [],          // Array para los colores
            tamanos: [],          // Array para los tamaños
            longitudes: [],       // Array para las longitudes
            unidadesMedida: [],   // Array para las unidades de medida
            ubicaciones: [],      // Array para las ubicaciones
            proveedores: [],      // Array para los proveedores
            imagenesPrevias: [],
            imagenesInfo: [],
            imagenesSeleccionadas: [],
            imagenesAEliminar: [],
            erroresValidacion: {},
            dragging: false,
            mostrarFormulario: false,
            modo: 'crear',
            colorSeleccionado: null,
            tamanoSeleccionado: null,
            longitudSeleccionada: null,
        };
    },
    computed: {
        esFormularioValido() {
            return this.producto.nombre &&
                this.producto.descripcion &&
                this.producto.categoria_producto_id &&
                this.producto.unidad_medida_id &&
                this.producto.ubicacion_id;
        },
        variantesSonValidas() {
            return this.variantes.every(variante => {
                return variante.color_id || variante.tamano_id || variante.longitud_id;
            });
        }
    },
    methods: {
        //Validar
        validarVariantes() {
            this.erroresValidacion.variantes = [];
            this.variantes.forEach((variante, index) => {
                if (!variante.color_id && !variante.tamano_id && !variante.longitud_id) {
                    this.erroresValidacion.variantes[index] = 'Debe seleccionar al menos color, tamaño o longitud';
                }
            });
            return this.erroresValidacion.variantes.length === 0;
        },
        // Métodos para obtener datos
        async cargarInventario() {
            try {
                const response = await axios.get(`/api/inventario/producto/${this.producto.id}`);
                this.variantes = response.data.map(item => ({
                    color_id: item.color_id,
                    tamano_id: item.tamano_id,
                    longitud_id: item.longitud_id,
                    precio_unitario: item.precio_unitario,
                    cantidad: item.cantidad
                }));
            } catch (error) {
                console.error('Error al cargar el inventario:', error);
            }
        },
        async obtenerOpciones() {
            await Promise.all([
                this.obtenerColores(),
                this.obtenerTamanos(),
                this.obtenerLongitudes(),
                this.obtenerCategorias(),
                this.obtenerUnidadesMedida(),
                this.obtenerUbicaciones(),
                this.obtenerProveedores()
            ]);
        },
        async obtenerColores() {
            try {
                const response = await axios.get('/api/colores');
                this.colores = response.data;
            } catch (error) {
                console.error('Error al obtener los colores:', error);
            }
        },
        async obtenerLongitudes() {
            try {
                const response = await axios.get('/api/longitudes');
                this.longitudes = response.data;
            } catch (error) {
                console.error('Error al obtener las longitudes:', error);
            }
        },
        async obtenerTamanos() {
            try {
                const response = await axios.get('/api/tamanos');
                this.tamanos = response.data;
            } catch (error) {
                console.error('Error al obtener los tamaños:', error);
            }
        },
        async obtenerCategorias() {
            try {
                const response = await axios.get('/api/categoriaproductos'); // Asegúrate de que la ruta sea correcta
                this.categorias = response.data;
            } catch (error) {
                console.error('Error al obtener las categorías:', error);
            }
        },
        async obtenerUnidadesMedida() {
            try {
                const response = await axios.get('/api/unidadesmedidas');
                this.unidadesMedida = response.data;
            } catch (error) {
                console.error('Error al obtener las unidades de medida:', error);
            }
        },
        async obtenerProveedores() {
            try {
                const response = await axios.get('/api/proveedores');
                this.proveedores = response.data;
            } catch (error) {
                console.error('Error al obtener los proveedores:', error);
            }
        },
        async obtenerUbicaciones() {
            try {
                const response = await axios.get('/api/ubicaciones'); // Asegúrate de que la ruta sea correcta
                this.ubicaciones = response.data;
            } catch (error) {
                console.error('Error al obtener las ubicaciones:', error);
            }
        },
        getCategoriaNombre(id) {
            const categoria = this.categorias.find(c => c.id === id);
            return categoria ? categoria.descripcion : 'Desconocida';
        },
        getUnidadMedidaNombre(id) {
            const unidad = this.unidadesMedida.find(u => u.id === id);
            return unidad ? unidad.descripcion : 'Desconocida';
        },
        getProveedorNombre(id) {
            const proveedor = this.proveedores.find(p => p.id === id);
            return proveedor ? proveedor.razon_social : 'Desconocida';
        },
        getUbicacionNombre(id) {
            const ubicacion = this.ubicaciones.find(u => u.id === id);
            return ubicacion ? ubicacion.descripcion : 'Desconocida';
        },
        mostrarCampo(tipo) {
            const categoriaSeleccionada = this.categorias.find(
                (categoria) => categoria.id === this.producto.categoria_producto_id
            );
            if (!categoriaSeleccionada) return false;

            // Ajusta las condiciones según las categorías que deben mostrar cada campo
            if (tipo === 'color' && categoriaSeleccionada.descripcion === 'Ropa') {
                return true;
            } else if (tipo === 'color' && categoriaSeleccionada.descripcion === 'Articulos') {
                return true;

            } else if (tipo === 'color' && categoriaSeleccionada.descripcion === 'Polos') {
                return true;
            } else if (tipo === 'color' && categoriaSeleccionada.descripcion === 'Collares') {
                return true;
            } else if (tipo === 'longitud' && categoriaSeleccionada.descripcion === 'Collares') {
                return true;
            } else if (tipo === 'tamano' && categoriaSeleccionada.descripcion === 'Ropa') {
                return true;
            }
            else if (tipo === 'longitud' && categoriaSeleccionada.descripcion === 'Articulos') {
                return true;
            }
            return false;
        },
        limpiarFormulario() {
            this.producto = {
                nombre: '',
                descripcion: '',
                categoria_producto_id: null,
                unidad_medida_id: null,
                ubicacion_id: null,
                estado: 'Activo',
                colores: [],
                tamanos: [],
                longitudes: [],
            };
            this.variantes = [{
                color_id: null,
                tamano_id: null,
                longitud_id: null,
                precio_unitario: 0,
                cantidad: 0
            }];
            this.imagenesSeleccionadas = [];
            this.imagenesPrevias = [];
            this.imagenesInfo = [];
            this.imagenesAEliminar = [];
        },
        procesarImagenes(event) {
            const archivos = event.target.files;
            Array.from(archivos).forEach((archivo) => {
                const lector = new FileReader();
                lector.onload = (e) => {
                    this.imagenesPrevias.push(e.target.result); // Acumula en el array
                };
                lector.readAsDataURL(archivo);
                this.imagenesSeleccionadas.push(archivo); // Solo agrega la nueva imagen
            });
        },
        eliminarImagen(index) {
            // Si hay información de imagen en ese índice, guardar el ID para eliminar
            if (this.imagenesInfo[index] && this.imagenesInfo[index].id) {
                this.imagenesAEliminar.push(this.imagenesInfo[index].id);
            }

            // Eliminar la imagen de las vistas previas
            this.imagenesPrevias.splice(index, 1);

            // Eliminar la información de la imagen
            this.imagenesInfo.splice(index, 1);

            // Si es una imagen nueva, eliminarla de las seleccionadas
            if (index >= this.imagenesInfo.length) {
                const newImageIndex = index - this.imagenesInfo.length;
                if (newImageIndex >= 0) {
                    this.imagenesSeleccionadas.splice(newImageIndex, 1);
                }
            }
        },
        handleDragOver(event) {
            event.preventDefault(); // Permitir el arrastre
            this.dragging = true; // Indicar que se está arrastrando
        },
        handleDrop(event) {
            this.dragging = false; // Restablecer el estado de arrastre
            const archivos = event.dataTransfer.files;
            Array.from(archivos).forEach((archivo) => {
                const lector = new FileReader();
                lector.onload = (e) => {
                    this.imagenesPrevias.push(e.target.result); // Acumula en el array
                };
                lector.readAsDataURL(archivo);
                this.imagenesSeleccionadas.push(archivo); // Almacena el archivo para envío
            });
        },
        abrirBiblioteca() {
            document.getElementById('imagenes').click(); // Abre el selector de archivos
        },
        toggleEstado() {
            // Cambia directamente el estado basado en el valor actual
            this.producto.estado = this.producto.estado === 'Activo' ? 'Desactivado' : 'Activo';
            console.log('Nuevo estado:', this.producto.estado); // Para debugging
        },
        dataURLtoFile(dataURL, filename) {
            const arr = dataURL.split(','),
                mime = arr[0].match(/:(.*?);/)[1],
                bstr = atob(arr[1]),
                byteLength = bstr.length, // Esto sigue siendo una constante.
                u8arr = new Uint8Array(byteLength);

            // Cambiar la declaración a let para permitir la modificación
            let index = byteLength;
            while (index--) {
                u8arr[index] = bstr.charCodeAt(index); // Usamos 'index' en lugar de 'byteLength'
            }

            return new File([u8arr], filename, { type: mime });
        },
        formatearCaracteristicas(array) {
            if (!array || array.length === 0) return '-';
            return array.map(item => item.descripcion).join(', ');
        },
        agregarVariante() {
            this.variantes.push({
                color_id: null,
                tamano_id: null,
                longitud_id: null,
                precio_unitario: 0,
                cantidad: 0
            });
        },

        eliminarVariante(index) {
            this.variantes.splice(index, 1);
        },
        async guardarProducto() {
            try {
                if (!this.validarVariantes()) {
                    this.error = 'Por favor, completa las variantes correctamente.';
                    return;
                }
                this.error = null;
                const data = new FormData();

                // Agregar los campos básicos del producto
                Object.keys(this.producto).forEach(key => {
                    if (key !== 'imagenes' && key !== 'colores' && key !== 'tamanos' && key !== 'longitudes') {
                        data.append(key, this.producto[key]);
                    }
                });

                console.log('Datos de variantes antes de enviar:', this.variantes);

                data.append('variantes', JSON.stringify(this.variantes));

                // Agregar las imágenes
                if (this.imagenesSeleccionadas) {
                    this.imagenesSeleccionadas.forEach((imagen) => {
                        data.append('imagenes[]', imagen);
                    });
                }

                // Agregar imágenes a eliminar
                if (this.imagenesAEliminar.length > 0) {
                    this.imagenesAEliminar.forEach(id => {
                        data.append('imagenes_a_eliminar[]', id);
                    });
                }

                const method = this.modo === 'crear' ? 'post' : 'post';
                const url = this.modo === 'crear'
                    ? '/api/productos'
                    : `/api/productos/${this.producto.id}?_method=PUT`;

                await axios[method](url, data, {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    },
                });

                // Emitir evento para notificar al componente padre
                this.$emit('productos-actualizados');
                this.$emit('guardar');
                this.$emit('cerrar');

                this.limpiarFormulario();
                this.erroresValidacion = {};

            } catch (error) {
                console.error('Error al guardar el producto:', error.response ? error.response.data : error.message);
                if (error.response && error.response.status === 422) {
                    this.erroresValidacion = error.response.data.errors;
                }
            }
        },
    },
    created() {
        console.log("Producto a editar:", this.productoEditado);

        if (this.productoEditado) {
            this.modo = 'editar';
            this.producto = { ...this.productoEditado };

            this.cargarInventario(); // Nuevo método para cargar el inventario

            // Inicializar las imágenes existentes
            if (this.productoEditado.imagenes) {
                this.imagenesInfo = [...this.productoEditado.imagenes];
                this.imagenesPrevias = this.productoEditado.imagenes.map(img =>
                    `http://localhost:8000/${img.url}`
                );
            }
        }
        this.obtenerOpciones();
    }

};
</script>

<template>
    <div class="container mt-0">
        <h2 class="mb-4 text-center">{{ productoEditado ? 'Editar' : 'Registrar' }} Producto</h2>

        <!-- Alert for errors -->
        <div v-if="error" class="alert alert-danger alert-dismissible fade show" role="alert">
            {{ error }}
            <button type="button" class="btn-close" @click="error = null"></button>
        </div>
        <form @submit.prevent="guardarProducto">
            <div class="form-group mb-3">
                <label for="nombre">Nombre</label>
                <input v-model="producto.nombre" type="text" id="nombre" class="form-control" required minlength="4"
                    maxlength="100">
            </div>
            <div class="form-group mb-3">
                <label for="descripcion">Descripción</label>
                <textarea v-model="producto.descripcion" id="descripcion" class="form-control" required
                    rows="4"></textarea>
            </div>
            <div class="form-group mb-3">
                <label for="categoria">Categoría</label>
                <select v-model="producto.categoria_producto_id" id="categoria" class="form-control" required>
                    <option v-for="categoria in categorias" :key="categoria.id" :value="categoria.id">
                        {{ categoria.descripcion }}
                    </option>
                </select>
            </div>

            <!-- Sección de Variantes -->
            <div v-if="producto.categoria_producto_id" class="form-group mb-4">
                <h4>Variantes del Producto</h4>

                <div v-for="(variante, index) in variantes" :key="index" class="border p-3 mb-3 rounded">
                    <!-- Selector de Color -->
                    <div v-if="mostrarCampo('color')" class="form-group mb-2">
                        <label>Color</label>
                        <select v-model="variante.color_id" class="form-control">
                            <option value="">Seleccione un color</option>
                            <option v-for="color in colores" :key="color.id" :value="color.id">
                                {{ color.descripcion }}
                            </option>
                        </select>
                    </div>

                    <!-- Selector de Tamaño -->
                    <div v-if="mostrarCampo('tamano')" class="form-group mb-2">
                        <label>Tamaño</label>
                        <select v-model="variante.tamano_id" class="form-control">
                            <option value="">Seleccione un tamaño</option>
                            <option v-for="tamano in tamanos" :key="tamano.id" :value="tamano.id">
                                {{ tamano.descripcion }}
                            </option>
                        </select>
                    </div>

                    <!-- Selector de Longitud -->
                    <div v-if="mostrarCampo('longitud')" class="form-group mb-2">
                        <label>Longitud</label>
                        <select v-model="variante.longitud_id" class="form-control">
                            <option value="">Seleccione una longitud</option>
                            <option v-for="longitud in longitudes" :key="longitud.id" :value="longitud.id">
                                {{ longitud.descripcion }}
                            </option>
                        </select>
                    </div>

                    <!-- Precio y Cantidad -->
                    <div class="row">
                        <div class="col-md-6">
                            <div class="form-group">
                                <label>Precio Unitario</label>
                                <input type="number" v-model="variante.precio_unitario" class="form-control" step="0.01"
                                    min="0" maxlength="10">
                            </div>
                        </div>
                        <div class="col-md-6">
                            <div class="form-group">
                                <label>Cantidad</label>
                                <input type="number" v-model="variante.cantidad" class="form-control" min="0">
                            </div>
                        </div>
                    </div>
                    <div v-if="erroresValidacion.variantes && erroresValidacion.variantes[index]" class="text-danger">
                        {{ erroresValidacion.variantes[index] }}
                    </div>
                    <!-- Botón para eliminar variante -->
                    <button type="button" class="btn btn-danger mt-2" @click="eliminarVariante(index)">
                        Eliminar Variante
                    </button>
                </div>

                <!-- Botón para agregar nueva variante -->
                <button type="button" class="btn btn-primary mt-2" @click="agregarVariante">
                    Agregar Variante
                </button>
            </div>

            <!-- Otros campos del formulario -->
            <div class="form-group">
                <label for="unidadMedida">Unidad de Medida</label>
                <select v-model="producto.unidad_medida_id" id="unidadMedida" class="form-control" required>
                    <option v-for="unidad in unidadesMedida" :key="unidad.id" :value="unidad.id">
                        {{ unidad.descripcion }}
                    </option>
                </select>
            </div>
            <div class="form-group">
                <label for="proveedor">Proveedor</label>
                <select v-model="producto.proveedor_id" id="proveedor" class="form-control" required>
                    <option v-for="proveedor in proveedores" :key="proveedor.id" :value="proveedor.id">
                        {{ proveedor.razon_social }}
                    </option>
                </select>
            </div>
            <div class="form-group">
                <label for="ubicacion">Ubicación</label>
                <select v-model="producto.ubicacion_id" id="ubicacion" class="form-control" required>
                    <option v-for="ubicacion in ubicaciones" :key="ubicacion.id" :value="ubicacion.id">
                        {{ ubicacion.descripcion }}
                    </option>
                </select>
            </div>
            <div class="form-group mb-3">
                <label for="imagenes">Imágenes</label>
                <div @dragover.prevent="handleDragOver" @drop.prevent="handleDrop" @dragenter="dragging = true"
                    @dragleave="dragging = false" @click="abrirBiblioteca"
                    class="drop-area border border-dashed p-5 text-center mb-2">
                    <p v-if="!dragging">Arrastra y suelta imágenes aquí o haz clic para seleccionar</p>
                    <p v-else class="text-success">Suelta la imagen aquí</p>
                    <input type="file" id="imagenes" @change="procesarImagenes" class="form-control" multiple
                        style="display: none;">
                </div>
                <div v-if="imagenesPrevias.length" class="mt-2">
                    <div v-for="(imagen, index) in imagenesPrevias" :key="index"
                        class="img-thumbnail-wrapper position-relative">
                        <img :src="imagen" alt="Vista previa" class="img-thumbnail" style="max-width: 200px;">
                        <button @click="eliminarImagen(index)" class="btn btn-danger btn-sm position-absolute"
                            style="top: 0; right: 0;">
                            &times;
                        </button>
                    </div>
                </div>
            </div>
            <div class="form-group">
                <label for="estadoToggle">Estado</label>
                <div class="form-check form-switch">
                    <input type="checkbox" class="form-check-input" id="estadoToggle"
                        :checked="producto.estado === 'Activo'" @change="toggleEstado">
                    <label class="form-check-label" for="estadoToggle">
                        {{ producto.estado }}
                    </label>
                </div>
            </div>
            <br>
            <div class="form-buttons">
                <button type="button" class="btn btn-secondary" @click="$emit('cerrar')">
                    Cancelar
                </button>
                <button type="submit" class="btn btn-success" :disabled="!esFormularioValido">
                    {{ productoEditado ? 'Actualizar' : 'Registrar' }} Producto
                </button>
            </div>
        </form>
    </div>
</template>