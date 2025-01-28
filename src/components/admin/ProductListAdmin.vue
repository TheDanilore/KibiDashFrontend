<script>
import AdminMenu from './AdminMenu.vue';
import ProductoForm from './ProductoForm.vue';
import axios from 'axios';

export default {
    components: {
        AdminMenu,
        ProductoForm,
    },
    computed: {
        // Método para obtener el inventario de un producto
        getInventarioAtributos() {
            return (productoId) => {
                return this.inventarioProductos[productoId] || {
                    colores: [],
                    tamanos: [],
                    longitudes: []
                };
            };
        }
    },
    data() {
        return {
            productos: [],
            categorias: [],
            unidadesMedida: [],
            ubicaciones: [],
            proveedores: [],
            colores: [],
            tamanos: [],
            longitudes: [],
            inventarioProductos: {},
            mostrarFormulario: false,
            productoEditado: null,
            activeIndex: {},
        };
    },
    methods: {
        async obtenerProductos() {
            try {
                const response = await axios.get('/api/productos');
                this.productos = response.data.data.map(producto => {
                    return {
                        ...producto,
                        colores: producto.colores || [],
                        tamanos: producto.tamanos || [],
                        longitudes: producto.longitudes || [],
                        imagenes: producto.imagenes || []
                    };
                });

                // Obtener el inventario para cada producto
                await Promise.all(this.productos.map(producto => this.obtenerInventarioProducto(producto.id)));

                // Inicializa el índice activo para cada producto
                this.productos.forEach(producto => {
                    this.activeIndex[producto.id] = 0;
                });
            } catch (error) {
                console.error('Error al obtener los productos:', error);
            }
        },
        // método para obtener el inventario específico de un producto
        async obtenerInventarioProducto(productoId) {
            try {
                const response = await axios.get(`/api/inventario/producto/${productoId}`);
                // Agrupar los atributos únicos del inventario
                const inventario = response.data;

                this.inventarioProductos[productoId] = {
                    colores: [...new Set(inventario.map(item => item.color_id))],
                    tamanos: [...new Set(inventario.map(item => item.tamano_id))],
                    longitudes: [...new Set(inventario.map(item => item.longitud_id))]
                };
            } catch (error) {
                console.error(`Error al obtener el inventario del producto ${productoId}:`, error);
                this.inventarioProductos[productoId] = {
                    colores: [],
                    tamanos: [],
                    longitudes: []
                };
            }
        },
        // Métodos helper para obtener las descripciones
        getColorDescripcion(colorId) {
            const color = this.colores.find(c => c.id === colorId);
            return color ? color.descripcion : '';
        },
        getTamanoDescripcion(tamanoId) {
            const tamano = this.tamanos.find(t => t.id === tamanoId);
            return tamano ? tamano.descripcion : '';
        },
        getLongitudDescripcion(longitudId) {
            const longitud = this.longitudes.find(l => l.id === longitudId);
            return longitud ? longitud.descripcion : '';
        },
        // Métodos para obtener datos
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
        nextImage(productoId) {
            if (this.productos.length > 0) {
                const producto = this.productos.find(p => p.id === productoId);
                if (producto) {
                    this.activeIndex[productoId] = (this.activeIndex[productoId] + 1) % producto.imagenes.length;
                }
            }
        },
        prevImage(productoId) {
            if (this.productos.length > 0) {
                const producto = this.productos.find(p => p.id === productoId);
                if (producto) {
                    this.activeIndex[productoId] =
                        (this.activeIndex[productoId] - 1 + producto.imagenes.length) % producto.imagenes.length;
                }
            }
        },
        // Métodos de gestión del formulario
        mostrarFormularioCreacion() {
            this.productoEditado = null;
            this.mostrarFormulario = true;
        },
        mostrarFormularioEdicion(producto) {
            this.productoEditado = {
                ...JSON.parse(JSON.stringify(producto)),
                colores: this.inventarioProductos[producto.id]?.colores || [],
                tamanos: this.inventarioProductos[producto.id]?.tamanos || [],
                longitudes: this.inventarioProductos[producto.id]?.longitudes || [],
            };
            this.mostrarFormulario = true;
        },
        async onGuardar() {
            // Este método ahora solo maneja la actualización de la lista y cierre del formulario
            await this.obtenerProductos();
            this.cerrarFormulario();
        },
        async eliminarProducto(id) {
            if (confirm('¿Estás seguro de que deseas eliminar este producto?')) {
                try {
                    await axios.delete(`/api/productos/${id}`);
                    await this.obtenerProductos();
                } catch (error) {
                    console.error('Error al eliminar el producto:', error);
                }
            }
        },
        cerrarFormulario() {
            this.mostrarFormulario = false;
            this.productoEditado = null;
        },
    },

    //cargar listas 
    mounted() {
        this.obtenerProductos();
        this.obtenerOpciones();
    }
};
</script>

<template>
    <div class="container mt-0">
        <AdminMenu />
        <br>
        <h2 class="mb-4 text-center">Lista de Productos</h2>
        <div class="table-responsive">
            <table class="table table-striped table-bordered text-center">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Nombre</th>
                        <th>Categoría</th>
                        <th>Colores</th>
                        <th>Longitudes</th>
                        <th>Tamaños</th>
                        <th>Unidad de Medida</th>
                        <th>Proveedor</th>
                        <th>Precio</th>
                        <th>Stock</th>
                        <th>Ubicación</th>
                        <th>Imagenes</th>
                        <th>Estado</th>
                        <th>Acciones</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="producto in productos" :key="producto.id">
                        <td>{{ producto.id }}</td>
                        <td>{{ producto.nombre }}</td>
                        <td>{{ getCategoriaNombre(producto.categoria_producto_id) }}</td>
                        <td>
                            <template v-if="getInventarioAtributos(producto.id).colores.length">
                                <span v-for="colorId in getInventarioAtributos(producto.id).colores" :key="colorId"
                                    class="badge bg-secondary me-1">
                                    {{ getColorDescripcion(colorId) }}
                                </span>
                            </template>
                            <span v-else>-</span>
                        </td>
                        <td>
                            <template v-if="getInventarioAtributos(producto.id).longitudes.length">
                                <span v-for="longitudId in getInventarioAtributos(producto.id).longitudes"
                                    :key="longitudId" class="badge bg-secondary me-1">
                                    {{ getLongitudDescripcion(longitudId) }}
                                </span>
                            </template>
                            <span v-else>-</span>
                        </td>
                        <td>
                            <template v-if="getInventarioAtributos(producto.id).tamanos.length">
                                <span v-for="tamanoId in getInventarioAtributos(producto.id).tamanos" :key="tamanoId"
                                    class="badge bg-info me-1">
                                    {{ getTamanoDescripcion(tamanoId) }}
                                </span>
                            </template>
                            <span v-else>-</span>
                        </td>
                        <td>{{ getUnidadMedidaNombre(producto.unidad_medida_id) }}</td>
                        <td>{{ getProveedorNombre(producto.proveedor_id) }}</td>
                        <td>S/{{ producto.precio_unitario_maximo.toFixed(2) || 0 }}</td>
                        <td>
                            <template v-if="producto.total_stock >= 500">
                                <span class="badge bg-success me-1">
                                    {{ producto.total_stock || 0 }}
                                </span>
                            </template>
                            <template v-if="producto.total_stock < 500 && producto.total_stock > 50">
                                <span class="badge bg-primary me-1">
                                    {{ producto.total_stock || 0 }}
                                </span>
                            </template>
                            <template v-if="producto.total_stock <= 50 && producto.total_stock > 10">
                                <span class="badge bg-info me-1">
                                    {{ producto.total_stock || 0 }}
                                </span>
                            </template>
                            <template v-if="producto.total_stock <= 10">
                                <span class="badge bg-danger me-1">
                                    {{ producto.total_stock || 0 }}
                                </span>
                            </template>
                        </td>
                        <td>{{ getUbicacionNombre(producto.ubicacion_id) }}</td>
                        <td>
                            <div v-if="producto.imagenes.length > 0">
                                <div class="carousel-container">
                                    <img :src="`http://localhost:8000/${producto.imagenes[activeIndex[producto.id]]?.url}`"
                                        class="d-block w-100" alt="Imagen del producto"
                                        style="max-height: 100px; object-fit: cover;" />
                                    <button @click="prevImage(producto.id)" class="carousel-control-prev">◀</button>
                                    <button @click="nextImage(producto.id)" class="carousel-control-next">▶</button>
                                </div>
                            </div>
                            <div v-else>
                                <p>No hay imágenes disponibles.</p>
                            </div>
                        </td>
                        <td>
                            <span v-if="producto.estado === 'Activo'" class="badge bg-success">Activo</span>
                            <span v-else class="badge bg-danger">Inactivo</span>
                        </td>
                        <td>
                            <button @click="mostrarFormularioEdicion(producto)" class="btn btn-primary btn-sm">
                                Editar
                            </button>
                            <button @click="eliminarProducto(producto.id)"
                                class="btn btn-danger btn-sm">Eliminar</button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="mt-4 text-center">
            <button @click="mostrarFormularioCreacion" class="btn btn-success">Añadir Nuevo Producto</button>
        </div>

        <!-- Formulario para crear/editar producto -->
        <ProductoForm v-if="mostrarFormulario" :productoEditado="productoEditado" :categorias="categorias"
            :unidades-medida="unidadesMedida" :ubicaciones="ubicaciones" :proveedores="proveedores" :colores="colores"
            :tamanos="tamanos" :longitudes="longitudes" @guardar="onGuardar" @cerrar="cerrarFormulario" />
    </div>
</template>

<style scoped>
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

h2 {
    font-size: 1.8rem;
    margin-bottom: 20px;
    color: #343a40;
}

.table-responsive {
    margin-top: 1.5rem;
    overflow-x: auto;
}

td {
    max-width: 200px;
    white-space: normal;
    word-wrap: break-word;
}

.table th,
.table td {
    vertical-align: middle;
    text-align: center;
}

.carousel-container {
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
}

.carousel-container img {
    max-height: 100px;
    object-fit: cover;
}

.carousel-control-prev,
.carousel-control-next {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    font-size: 1.5rem;
    cursor: pointer;
    background-color: rgba(0, 0, 0, 0.5);
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    z-index: 10;
}

.carousel-control-prev {
    left: 5px;
}

.carousel-control-next {
    right: 5px;
}
</style>