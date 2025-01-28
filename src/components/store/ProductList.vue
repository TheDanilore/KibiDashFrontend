<script>
import axios from 'axios';

export default {
    data() {
        return {
            productos: [],
            activeIndex: {},
            loading: null,
            error: null,
        };
    },
    watch: {
        '$route.query.categoria': {
            immediate: true,
            handler(newCategoria) {
                this.obtenerProductos(newCategoria);
            }
        }
    },
    methods: {
        async obtenerProductos(categoriaId = null) {
            this.loading = true;
            this.error = null;

            try {
                const response = await axios.get('/api/productos', {
                    params: {
                        categoria: categoriaId,
                        estado: 'Activo'
                    }  // Agregar el parámetro de categoría
                });

                const productos = response.data.data;

                // Procesamiento de productos e inventarios...
                for (const producto of productos) {
                    const inventarioResponse = await axios.get(`/api/inventario/producto/${producto.id}`);
                    let inventario = inventarioResponse.data;

                    inventario = inventario.reduce((acc, variante) => {
                        const existeVariante = acc.some(item =>
                            item.color_id === variante.color_id &&
                            item.tamano_id === variante.tamano_id &&
                            item.longitud_id === variante.longitud_id
                        );

                        if (!existeVariante) {
                            acc.push(variante);
                        }
                        return acc;
                    }, []);

                    producto.opcionesUnicas = {
                        colores: [...new Set(inventario.map(v => v.color).filter(Boolean))],
                        tamanos: [...new Set(inventario.map(v => v.tamano).filter(Boolean))],
                        longitudes: [...new Set(inventario.map(v => v.longitud).filter(Boolean))]
                    };

                    producto.inventario = inventario;
                }

                this.productos = productos;
                this.productos.forEach(producto => {
                    this.activeIndex[producto.id] = 0;
                });
            } catch (error) {
                console.error('Error al obtener los productos:', error);
                this.error = 'Error al obtener los productos. Inténtelo de nuevo más tarde.';
            } finally {
                this.loading = false;
            }
        },
        tieneUnicaVariante(producto) {
            return producto.inventario.length === 1;
        },
        tieneMultiplesVariantes(producto) {
            return producto.inventario.length > 1;
        },
        nextImage(productoId) {
            const producto = this.productos.find(p => p.id === productoId);
            if (producto && producto.imagenes.length > 0) {
                this.activeIndex[productoId] = (this.activeIndex[productoId] + 1) % producto.imagenes.length;
            }
        },
        prevImage(productoId) {
            const producto = this.productos.find(p => p.id === productoId);
            if (producto && producto.imagenes.length > 0) {
                this.activeIndex[productoId] =
                    (this.activeIndex[productoId] - 1 + producto.imagenes.length) % producto.imagenes.length;
            }
        },
        verDetalle(id) {
            this.$router.push(`/productos/${id}`);
        },
        añadirAlCarrito(producto) {
            // Verificar que el producto tiene una única variante
            if (!this.tieneUnicaVariante(producto)) {
                this.verDetalle(producto.id);
                return;
            }

            // Obtener la única variante del inventario
            const variante = producto.inventario[0];

            // Verificar si hay stock disponible
            if (!variante || variante.cantidad <= 0) {
                alert("Lo sentimos, el producto está agotado");
                return;
            }

            // Crear el item para el carrito
            const cartItem = {
                id: producto.id,
                inventario_id: variante.id,
                nombre: producto.nombre,
                precio: variante.precio_unitario,
                cantidad: 1,
                stock: variante.cantidad,
                imagen: producto.imagenes[0]?.url || "",
                color: variante.color || null,
                tamano: variante.tamano || null,
                longitud: variante.longitud || null,
            };

            // Obtener el carrito actual del localStorage
            const cart = JSON.parse(localStorage.getItem("cart") || "[]");

            // Buscar si el producto ya existe en el carrito
            const existingItemIndex = cart.findIndex(item =>
                item.id === cartItem.id &&
                item.inventario_id === cartItem.inventario_id
            );

            // Si el producto ya existe, actualizar cantidad
            if (existingItemIndex !== -1) {
                // Verificar que no exceda el stock disponible
                if (cart[existingItemIndex].cantidad + 1 > variante.cantidad) {
                    alert("No hay suficiente stock disponible");
                    return;
                }
                cart[existingItemIndex].cantidad += 1;
            } else {
                // Si no existe, agregar el nuevo item
                cart.push(cartItem);
            }

            // Guardar el carrito actualizado
            localStorage.setItem("cart", JSON.stringify(cart));
            alert("¡Producto añadido al carrito!");
        }
    },
    mounted() {
        const categoriaId = this.$route.query.categoria;
        this.obtenerProductos(categoriaId);
    },
};
</script>

<template>
    <div class="product-list">
        <h2 class="title">Nuestros Productos</h2>
        <div v-if="loading" class="text-center py-5">
            <div class="spinner-border" role="status">
                <span class="visually-hidden">Cargando...</span>
            </div>
        </div>

        <div v-else-if="error" class="alert alert-danger" role="alert">
            {{ error }}
        </div>
        <div class="product-grid">
            <div class="product-card" v-for="producto in productos" :key="producto.id">
                <div v-if="producto.imagenes && producto.imagenes.length > 0" class="carousel-container"
                    @click.self="verDetalle(producto.id)">

                    <img :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${producto.imagenes[activeIndex[producto.id]]?.url}`"
                        alt="Imagen del producto" />
                    <!-- <img :src="`http://localhost:8000/${producto.imagenes[activeIndex[producto.id]]?.url}`"
                        alt="Imagen del producto" /> -->
                    <button @click.stop="prevImage(producto.id)" class="carousel-control-prev">❮</button>
                    <button @click.stop="nextImage(producto.id)" class="carousel-control-next">❯</button>
                    <div class="image-counter">
                        {{ activeIndex[producto.id] + 1 }}/{{ producto.imagenes.length }}
                    </div>
                </div>
                <div class="card-body">
                    <h5 class="product-name">{{ producto.nombre }}</h5>
                    <p class="product-description">{{ producto.descripcion }}</p>
                    <p class="product-price">
                        S/{{ producto.inventario[0]?.precio_unitario || producto.precio_unitario_maximo }}
                    </p>
                    <!-- Mostrar información de variantes disponibles -->
                    <div class="variants-info" v-if="tieneMultiplesVariantes(producto)">
                        <p class="variant-text">Opciones disponibles:</p>
                        <ul class="variant-list">
                            <li v-if="producto.opcionesUnicas.colores.length > 0">
                                <span class="variant-icon">🎨</span>
                                Colores: {{ producto.opcionesUnicas.colores.join(', ') }}
                            </li>
                            <li v-if="producto.opcionesUnicas.tamanos.length > 0">
                                <span class="variant-icon">📏</span>
                                Tamaños: {{ producto.opcionesUnicas.tamanos.join(', ') }}
                            </li>
                            <li v-if="producto.opcionesUnicas.longitudes.length > 0">
                                <span class="variant-icon">📐</span>
                                Longitudes: {{ producto.opcionesUnicas.longitudes.join(', ') }}
                            </li>
                        </ul>
                    </div>

                    <div class="button-group">
                        <button v-if="tieneUnicaVariante(producto)" @click="añadirAlCarrito(producto)"
                            class="btn btn-cart" :disabled="!producto.inventario[0]?.cantidad">
                            <span class="btn-icon">🛒</span>
                            {{ producto.inventario[0]?.cantidad ? 'Añadir al carrito' : 'Agotado' }}
                        </button>
                        <button v-else @click="verDetalle(producto.id)" class="btn btn-options">
                            <span class="btn-icon">⚙️</span>
                            Seleccionar opciones
                        </button>
                        <button @click="verDetalle(producto.id)" class="btn btn-details">
                            <span class="btn-icon">ℹ️</span>
                            Ver detalle
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.product-list {
    max-width: auto;
    margin: 1.5rem auto;
    padding: 1.5rem;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.title {
    text-align: center;
    margin-bottom: 2rem;
    font-size: 1.75rem;
    color: #2c3e50;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    position: relative;
}

.title::after {
    content: '';
    position: absolute;
    bottom: -8px;
    left: 50%;
    transform: translateX(-50%);
    width: 315px;
    height: 3px;
    background: #237ddc;
    border-radius: 2px;
}

.product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
    gap: 1.25rem;
    padding: 0.5rem;
}

.product-card {
    background: #ffffff;
    border-radius: 8px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.06);
    overflow: hidden;
    transition: all 0.2s ease;
    position: relative;
}

.product-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.carousel-container {
    position: relative;
    width: 100%;
    height: 200px;
    overflow: hidden;
}

.carousel-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.3s ease;
}

.carousel-container:hover img {
    transform: scale(1.05);
}

.image-counter {
    position: absolute;
    bottom: 8px;
    right: 8px;
    background: rgba(0, 0, 0, 0.6);
    color: white;
    padding: 2px 6px;
    border-radius: 4px;
    font-size: 0.75rem;
}

.carousel-control-prev,
.carousel-control-next {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(255, 255, 255, 0.9);
    color: #333;
    border: none;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    cursor: pointer;
    opacity: 0;
    transition: opacity 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.9rem;
}

.carousel-container:hover .carousel-control-prev,
.carousel-container:hover .carousel-control-next {
    opacity: 1;
}

.carousel-control-prev {
    left: 8px;
}

.carousel-control-next {
    right: 8px;
}

.card-body {
    padding: 1rem;
}

.product-name {
    font-size: 1.1rem;
    color: #2c3e50;
    margin: 0.4rem 0;
    font-weight: 600;
    line-height: 1.3;
}

.product-description {
    font-size: 0.9rem;
    color: #666;
    margin: 0.75rem 0;
    line-height: 1.4;
}

.product-price {
    font-size: 1.2rem;
    font-weight: 700;
    color: #e74c3c;
    margin: 0.75rem 0;
}

.variants-info {
    margin: 0.75rem 0;
    padding: 0.75rem;
    background: #f8f9fa;
    border-radius: 6px;
    border: 1px solid #e9ecef;
}

.variant-text {
    color: #2c3e50;
    margin-bottom: 0.5rem;
    font-weight: 600;
    font-size: 0.85rem;
}

.variant-list {
    list-style: none;
    padding: 0;
    margin: 0;
}

.variant-list li {
    display: flex;
    align-items: center;
    margin: 0.4rem 0;
    color: #666;
    font-size: 0.85rem;
}

.variant-icon {
    margin-right: 0.4rem;
    font-size: 0.9rem;
}

.button-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    margin-top: 1rem;
}

.btn {
    width: 100%;
    padding: 0.6rem 1rem;
    border: none;
    border-radius: 6px;
    font-size: 0.9rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.4rem;
}

.btn-icon {
    font-size: 1rem;
}

.btn-cart {
    background: #28a745;
    color: white;
}

.btn-cart:hover {
    background: #218838;
    transform: translateY(-2px);
}

.btn-options {
    background: #6c757d;
    color: white;
}

.btn-options:hover {
    background: #5a6268;
    transform: translateY(-2px);
}

.btn-details {
    background: #007bff;
    color: white;
}

.btn-details:hover {
    background: #0056b3;
    transform: translateY(-2px);
}

@media (max-width: 768px) {
    .product-list {
        padding: 1rem;
        margin: 0.75rem;
        margin-top: 3rem;
        max-width: auto;
    }

    .title {
        font-size: 1.5rem;
        margin-bottom: 1.5rem;
    }

    .title::after {
        content: '';
        position: absolute;
        bottom: -8px;
        left: 50%;
        transform: translateX(-50%);
        width: 50px;
        height: 3px;
        background: #237ddc;
        border-radius: 2px;
    }

    .product-grid {
        gap: 1rem;
    }

    .carousel-container {
        height: 180px;
    }
}
</style>