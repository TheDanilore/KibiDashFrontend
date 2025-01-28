<script>
import { defineComponent, ref, computed, onMounted } from "vue";
import axios from "axios";
import { useRoute } from "vue-router";

export default defineComponent({
    name: "ProductDetail",
    setup() {
        const route = useRoute();
        const product = ref({
            imagenes: [],
            colores: [],
            tamanos: [],
            longitudes: [],
            inventarios: [],
            precio_unitario: 0
        });
        const selectedImage = ref("");
        const quantity = ref(1);
        const selectedColor = ref(null);
        const selectedSize = ref(null);
        const selectedLongitud = ref(null);
        const loading = ref(true);
        const error = ref(null);

        // Computed property para obtener el inventario seleccionado
        const selectedInventory = computed(() => {
            if (!product.value?.inventario) return null;

            return product.value.inventario.find(inv =>
                (!selectedColor.value || inv.color_id === selectedColor.value) &&
                (!selectedSize.value || inv.tamano_id === selectedSize.value) &&
                (!selectedLongitud.value || inv.longitud_id === selectedLongitud.value)
            );
        });

        // Computed property para el stock disponible de la variante seleccionada
        const availableStock = computed(() => {
            return selectedInventory.value?.cantidad || 0;
        });

        // Computed property para el precio de la variante seleccionada
        const selectedPrice = computed(() => {
            return selectedInventory.value?.precio_unitario || product.value.precio_unitario;
        });

        // Computed property para verificar si hay stock disponible
        const hasStock = computed(() => availableStock.value > 0);

        // Computed para filtrar inventarios según las selecciones actuales
        const filteredInventories = computed(() => {
            if (!product.value?.inventarios) return [];

            return product.value.inventario.filter(inv =>
                (!selectedColor.value || inv.color_id === selectedColor.value) &&
                (!selectedSize.value || inv.tamano_id === selectedSize.value) &&
                (!selectedLongitud.value || inv.longitud_id === selectedLongitud.value)
            );
        });

        /// Computed para obtener colores disponibles
        const availableColors = computed(() => {
            if (!product.value?.inventarios) return [];

            const colorIds = new Set();
            product.value.inventario.forEach(inv => {
                if (inv.cantidad > 0 &&
                    (!selectedSize.value || inv.tamano_id === selectedSize.value) &&
                    (!selectedLongitud.value || inv.longitud_id === selectedLongitud.value)) {
                    colorIds.add(inv.color_id);
                }
            });

            return product.value.colores.filter(color => colorIds.has(color.id));
        });

        // Computed para obtener tamaños disponibles
        const availableSizes = computed(() => {
            if (!product.value?.inventarios) return [];

            const sizeIds = new Set();
            product.value.inventario.forEach(inv => {
                if (inv.cantidad > 0 &&
                    (!selectedColor.value || inv.color_id === selectedColor.value) &&
                    (!selectedLongitud.value || inv.longitud_id === selectedLongitud.value)) {
                    sizeIds.add(inv.tamano_id);
                }
            });

            return product.value.tamanos.filter(size => sizeIds.has(size.id));
        });

        // Computed para obtener longitudes disponibles
        const availableLongitudes = computed(() => {
            if (!product.value?.inventarios) return [];

            const longitudIds = new Set();
            product.value.inventario.forEach(inv => {
                if (inv.cantidad > 0 &&
                    (!selectedColor.value || inv.color_id === selectedColor.value) &&
                    (!selectedSize.value || inv.tamano_id === selectedSize.value)) {
                    longitudIds.add(inv.longitud_id);
                }
            });

            return product.value.longitudes.filter(longitud => longitudIds.has(longitud.id));
        });

        const validateAndUpdateQuantity = (event) => {
            const value = parseInt(event.target.value);
            const maxQuantity = Math.min(availableStock.value, 50); // Límite: stock o 50

            if (isNaN(value) || value < 1) {
                quantity.value = 1; // Valor mínimo permitido
            } else if (value > maxQuantity) {
                quantity.value = maxQuantity; // Límite máximo
            } else {
                quantity.value = value; // Valor válido
            }
        };

        // Resetear cantidad al cambiar selecciones
        const resetQuantity = () => {
            quantity.value = 1;
        };

        // Manejadores de selección
        const handleColorSelect = (colorId) => {
            // Si ya está seleccionado, deseleccionar
            selectedColor.value = selectedColor.value === colorId ? null : colorId;
            resetQuantity();
        };

        const handleSizeSelect = (sizeId) => {
            // Si ya está seleccionado, deseleccionar
            selectedSize.value = selectedSize.value === sizeId ? null : sizeId;
            resetQuantity();
        };

        const handleLongitudSelect = (longitudId) => {
            // Si ya está seleccionado, deseleccionar
            selectedLongitud.value = selectedLongitud.value === longitudId ? null : longitudId;
            resetQuantity();
        };


        // Computed property para la validación de selección
        const isValidSelection = computed(() => {
            const needsColor = product.value.colores?.length > 0;
            const needsSize = product.value.tamanos?.length > 0;
            const needsLength = product.value.longitudes?.length > 0;

            const hasColor = selectedColor.value || !needsColor;
            const hasSize = selectedSize.value || !needsSize;
            const hasLength = selectedLongitud.value || !needsLength;

            return hasColor && hasSize && hasLength && hasStock.value;
        });

        // Mensaje de validación actualizado
        const validationMessage = computed(() => {
            if (!hasStock.value) return "Producto agotado";
            if (!isValidSelection.value) {
                const missing = [];
                if (product.value.colores?.length && !selectedColor.value) missing.push("color");
                if (product.value.tamanos?.length && !selectedSize.value) missing.push("tamaño");
                if (product.value.longitudes?.length && !selectedLongitud.value) missing.push("longitud");
                return `Por favor selecciona ${missing.join(", ")}`;
            }
            return "";
        });

        // Función para cargar los datos del producto
        const loadProductData = async () => {
            const productId = route.params.id;
            if (!productId) {
                error.value = "ID de producto no definido.";
                return;
            }

            try {
                loading.value = true;
                const response = await axios.get(`/api/productos/${productId}`);
                product.value = {
                    ...response.data,
                    imagenes: response.data.imagenes || [],
                    colores: response.data.colores || [],
                    tamanos: response.data.tamanos || [],
                    longitudes: response.data.longitudes || [],
                    inventarios: response.data.inventarios || [],
                    precio_unitario: response.data.precio_unitario || 0
                };
                if (product.value.imagenes.length > 0) {
                    selectedImage.value = product.value.imagenes[0]?.url || "";
                }
            } catch (err) {
                error.value = "Error al cargar los detalles del producto";
                console.error("Error al cargar los detalles del producto:", err);
            } finally {
                loading.value = false;
            }
        };

        // Función para manejar el zoom de la imagen
        const updateZoomPosition = (event) => {
            const img = event.target;
            const rect = img.getBoundingClientRect();
            const x = ((event.clientX - rect.left) / rect.width) * 100;
            const y = ((event.clientY - rect.top) / rect.height) * 100;
            img.style.transformOrigin = `${x}% ${y}%`;
        };

        const increaseQuantity = () => {
            // Obtén el límite máximo entre el stock disponible y 50
            const maxQuantity = Math.min(availableStock.value, 50);

            // Solo incrementa la cantidad si no ha alcanzado el límite
            if (quantity.value < maxQuantity) {
                quantity.value++;
            } else {
                alert(`La cantidad máxima permitida es ${maxQuantity}`);
            }
        };

        const decreaseQuantity = () => {
            if (quantity.value > 1) {
                quantity.value--;
            }
        };


        // Actualizar addToCart para incluir el inventario seleccionado
        const addToCart = () => {
            if (!isValidSelection.value || !selectedInventory.value) {
                alert('Por favor selecciona todas las opciones del producto');
                return;
            }

            const cart = JSON.parse(localStorage.getItem("cart") || "[]");
            const cartItem = {
                id: product.value.id,
                inventario_id: selectedInventory.value.id,
                nombre: product.value.nombre,
                precio: selectedPrice.value,
                cantidad: quantity.value,
                stock: selectedInventory.value.cantidad,
                imagen: selectedImage.value,
                color: product.value.colores.find(c => c.id === selectedColor.value)?.descripcion,
                tamano: product.value.tamanos.find(t => t.id === selectedSize.value)?.descripcion,
                longitud: product.value.longitudes.find(l => l.id === selectedLongitud.value)?.descripcion,
            };

            const existingItemIndex = cart.findIndex(item =>
                item.id === cartItem.id &&
                item.inventario_id === cartItem.inventario_id
            );

            if (existingItemIndex !== -1) {
                const newQuantity = cart[existingItemIndex].cantidad + quantity.value;
                if (newQuantity <= selectedInventory.value.cantidad) {
                    cart[existingItemIndex].cantidad = newQuantity;
                } else {
                    alert(`No hay suficiente stock disponible. Stock máximo: ${selectedInventory.value.cantidad}`);
                    return;
                }
            } else {
                cart.push(cartItem);
            }

            localStorage.setItem("cart", JSON.stringify(cart));
            alert("¡Producto añadido al carrito!");
            quantity.value = 1;
        };


        // Funciones para navegación de imágenes
        const prevImage = () => {
            const currentIndex = product.value.imagenes.findIndex(img => img.url === selectedImage.value);
            const newIndex = (currentIndex - 1 + product.value.imagenes.length) % product.value.imagenes.length;
            selectedImage.value = product.value.imagenes[newIndex].url;
        };

        const nextImage = () => {
            const currentIndex = product.value.imagenes.findIndex(img => img.url === selectedImage.value);
            const newIndex = (currentIndex + 1) % product.value.imagenes.length;
            selectedImage.value = product.value.imagenes[newIndex].url;
        };

        onMounted(loadProductData);


        const resetZoom = (event) => {
            event.target.style.removeProperty("--mouse-x");
            event.target.style.removeProperty("--mouse-y");
        };


        return {
            product,
            selectedImage,
            quantity,
            selectedColor,
            selectedSize,
            selectedLongitud,
            loading,
            error,
            hasStock,
            availableStock,
            selectedPrice,
            isValidSelection,
            validationMessage,
            availableColors,
            availableSizes,
            availableLongitudes,
            filteredInventories,
            validateAndUpdateQuantity,
            handleColorSelect,
            handleSizeSelect,
            handleLongitudSelect,
            increaseQuantity,
            decreaseQuantity,
            addToCart,
            prevImage,
            nextImage,
            updateZoomPosition,
            resetZoom
        };
    },
});
</script>

<template>
    <div class="product-detail">
        <div v-if="loading" class="text-center py-5">
            <div class="spinner-border" role="status">
                <span class="visually-hidden">Cargando...</span>
            </div>
        </div>

        <div v-else-if="error" class="alert alert-danger" role="alert">
            {{ error }}
        </div>

        <div v-else-if="product" class="row">
            <!-- Columna de imágenes -->
            <div class="col-md-6">
                <div class="image-gallery">
                    <div class="main-image position-relative">

                        <img :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${selectedImage}`" :alt="product.nombre"
                            class="img-fluid" @mousemove="updateZoomPosition" />
                        <!-- <img :src="`http://localhost:8000/${selectedImage}`" :alt="product.nombre" class="img-fluid"
                            @mousemove="updateZoomPosition" /> -->
                        <div class="navigation-arrows">
                            <button class="btn btn-light position-absolute start-0 top-50 translate-middle-y"
                                @click="prevImage" v-if="product.imagenes.length > 1">
                                ❮
                            </button>
                            <button class="btn btn-light position-absolute end-0 top-50 translate-middle-y"
                                @click="nextImage" v-if="product.imagenes.length > 1">
                                ❯
                            </button>
                        </div>
                    </div>
                    <div class="thumbnail-images d-flex gap-2 mt-3" v-if="product.imagenes.length > 1">

                        <img v-for="(imagen, index) in product.imagenes" :key="index"
                            :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${imagen.url}`"
                            :alt="`${product.nombre} - Vista ${index + 1}`" class="thumbnail cursor-pointer"
                            :class="{ 'active': selectedImage === imagen.url }" @click="selectedImage = imagen.url"
                            style="width: 80px; height: 80px; object-fit: cover;" />
                        <!-- <img v-for="(imagen, index) in product.imagenes" :key="index"
                            :src="`http://localhost:8000/${imagen.url}`" :alt="`${product.nombre} - Vista ${index + 1}`"
                            class="thumbnail cursor-pointer" :class="{ 'active': selectedImage === imagen.url }"
                            @click="selectedImage = imagen.url" style="width: 80px; height: 80px; object-fit: cover;" /> -->
                    </div>
                </div>
            </div>

            <!-- Columna de detalles -->
            <div class="col-md-6">
                <div class="detalle-producto mb-3">
                    <h1 class="product-title h2 mb-3">{{ product.nombre }}</h1>
                    <p class="product-price h3 text-primary mb-4">
                        S/ {{ selectedPrice }}
                    </p>
                    <p class="product-description mb-4">{{ product.descripcion }}</p>

                    <!-- Stock disponible -->
                    <div :class="['alert', hasStock ? 'alert-success' : 'alert-danger']">
                        <i :class="['bi', hasStock ? 'bi-check-circle-fill' : 'bi-x-circle-fill']"></i>
                        {{ hasStock ? `Stock disponible: ${availableStock} unidades` : 'Producto agotado' }}
                    </div>

                    <!-- Selección de colores -->
                    <div v-if="availableColors.length" class="mb-4">
                        <h4 class="h6 mb-2">Colores disponibles:</h4>
                        <div class="d-flex flex-wrap gap-2">
                            <button v-for="color in availableColors" :key="color.id" class="btn btn-outline-secondary"
                                :class="{ 'active': selectedColor === color.id }" @click="handleColorSelect(color.id)">
                                {{ color.descripcion }}
                            </button>
                        </div>
                    </div>

                    <!-- Selección de tamaños -->
                    <div v-if="availableSizes.length" class="mb-4">
                        <h4 class="h6 mb-2">Tamaños disponibles:</h4>
                        <div class="d-flex flex-wrap gap-2">
                            <button v-for="tamano in availableSizes" :key="tamano.id" class="btn btn-outline-secondary"
                                :class="{ 'active': selectedSize === tamano.id }" @click="handleSizeSelect(tamano.id)">
                                {{ tamano.descripcion }}
                            </button>
                        </div>
                    </div>

                    <!-- Selección de Longitudes -->
                    <div v-if="availableLongitudes.length" class="mb-4">
                        <h4 class="h6 mb-2">Longitudes disponibles:</h4>
                        <div class="d-flex flex-wrap gap-2">
                            <button v-for="longitud in availableLongitudes" :key="longitud.id"
                                class="btn btn-outline-secondary"
                                :class="{ 'active': selectedLongitud === longitud.id }"
                                @click="handleLongitudSelect(longitud.id)">
                                {{ longitud.descripcion }}
                            </button>
                        </div>
                    </div>

                    <!-- Control de cantidad -->
                    <div class="quantity-control mb-4">
                        <label class="h6 mb-2">Cantidad:</label>
                        <div class="d-flex align-items-center gap-2">
                            <button @click="quantity > 1 && quantity--" class="btn btn-outline-secondary"
                                :disabled="quantity <= 1">
                                -
                            </button>
                            <input type="number" v-model="quantity" @input="validateAndUpdateQuantity" min="1"
                                :max="Math.min(availableStock, 50)" class="form-control text-center"
                                style="width: 80px" />
                            <button @click="increaseQuantity" class="btn btn-outline-secondary"
                                :disabled="quantity >= Math.min(availableStock, 50)">
                                +
                            </button>
                        </div>
                    </div>

                    <!-- Botón de añadir al carrito -->
                    <button @click="addToCart" class="btn btn-primary btn-lg w-100 mb-2" :disabled="!isValidSelection">
                        <i class="bi bi-cart-plus"></i> Añadir al Carrito
                    </button>

                    <!-- Mensaje de validación -->
                    <p v-if="validationMessage" class="text-danger small text-center">
                        {{ validationMessage }}
                    </p>
                </div>

            </div>
        </div>
    </div>
</template>

<style scoped>
.main-image {
    overflow: hidden;
    cursor: zoom-in;
    max-width: 80%;
}


.main-image img {
    transition: transform 0.3s ease;
}

.main-image:hover img {
    transform: scale(1.5);
}

.thumbnail {
    cursor: pointer;
    transition: opacity 0.2s ease;
}

.thumbnail:hover {
    opacity: 0.8;
}

.thumbnail.active {
    border: 2px solid var(--bs-primary);
}

.navigation-arrows button {
    opacity: 0.7;
    transition: opacity 0.2s ease;
}

.navigation-arrows button:hover {
    opacity: 1;
}

.btn.active {
    background-color: var(--bs-primary);
    color: white;
    border-color: var(--bs-primary);
}



.product-detail {
    max-width: auto;
    margin: 0.5rem auto;
    padding: 1.5rem;
    background: #ffffff;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.image-gallery {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.thumbnail-images {
    display: flex;
    justify-content: center;
    margin-bottom: 15px;
}

.thumbnail-image {
    width: 60px;
    height: 60px;
    object-fit: cover;
    margin: 0 5px;
    cursor: pointer;
    border: 2px solid transparent;
    transition: border-color 0.3s, transform 0.3s;
}

.thumbnail-image:hover {
    transform: scale(1.1);
}

.thumbnail-image.active {
    border-color: #ff5c5c;
}



.displayed-image {
    width: 100%;
    height: auto;
    object-fit: cover;
    border-radius: 10px;
    transition: transform 0.3s ease;
    transform-origin: var(--mouse-x) var(--mouse-y);
}



/* Flechas de navegación, inicialmente ocultas */
.navigation-arrows {
    position: absolute;
    top: 50%;
    width: 100%;
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-between;
    transform: translateY(-50%);
    opacity: 0;
    transition: opacity 0.3s;
}

/* Mostrar flechas al pasar el cursor */
.main-image:hover .navigation-arrows {
    opacity: 1;
}

.arrow {
    background-color: rgba(255, 255, 255, 0.7);
    border: none;
    cursor: pointer;
    padding: 10px;
    border-radius: 50%;
    font-size: 20px;
    transition: background-color 0.3s;
}

.arrow:hover {
    background-color: rgba(255, 255, 255, 1);
}

.product-title {
    font-size: 2rem;
    font-weight: bold;
    color: #333;
    margin-bottom: 10px;
}

.product-price {
    font-size: 1.5rem;
    color: #ff5c5c;
    margin-bottom: 20px;
}

.product-description {
    font-size: 1rem;
    color: #666;
    margin-bottom: 20px;
}

/* Control de cantidad */
.quantity-control {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 20px;
}

.quantity-input {
    width: 50px;
    height: 30px;
    text-align: center;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin: 0 4px;
    padding: 0 4px;
    font-size: 14px;
}

.quantity-input::-webkit-inner-spin-button,
.quantity-input::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
}

.quantity-input:focus {
    outline: none;
    border-color: #666;
}

.quantity {
    font-size: 1.2rem;
    min-width: 60px;
    text-align: center;
}

.btn-primary {
    background-color: #ff5c5c;
    border: none;
    padding: 10px 20px;
}

.btn-primary:hover {
    background-color: #e04a4a;
}

.color-option,
.size-option,
.longitud-option {
    min-width: 40px;
    padding: 0.5rem 1rem;
}

.color-option.active,
.size-option.active,
.longitud-option.active {
    background-color: #007bff;
    color: white;
    border-color: #007bff;
}

@media (max-width: 768px) {
    .quantity {
        grid-area: quantity;
    }

}
</style>