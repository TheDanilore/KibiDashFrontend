<script setup>
import { ref, computed, onMounted, watch } from 'vue';

const cartItems = ref([]);

// Cargar los artículos del carrito desde localStorage
function loadCartItems() {
    cartItems.value = JSON.parse(localStorage.getItem('cart') || '[]');
}

// Comprobar si el precio es un número válido
function isValidprecio(precio) {
    return typeof precio === 'number' && !isNaN(precio);
}

function validateAndUpdateQuantity(event, index) {
    let value = parseInt(event.target.value);

    // Si no es un número o es menor que 1, establecer a 1
    if (isNaN(value) || value < 1) {
        cartItems.value[index].cantidad = 1;
    } else if (value > 50) {
        cartItems.value[index].cantidad = 50;
    }//Si la cantidad es mayor que el stock, establecer el stock
    else if (value > cartItems.value[index].stock) {
        cartItems.value[index].cantidad = cartItems.value[index].stock;
    }
    else {
        cartItems.value[index].cantidad = value;
    }

    localStorage.setItem('cart', JSON.stringify(cartItems.value));
}

// Aumentar la cantidad de un artículo
function increaseQuantity(index) {
    if (cartItems.value[index].cantidad >= 50) {
        console.log("La cantidad de este producto no puede ser mayor a 50");
        cartItems.value[index].cantidad = 50;
    } else {
        cartItems.value[index].cantidad++;
    }
    localStorage.setItem('cart', JSON.stringify(cartItems.value));
}

// Disminuir la cantidad de un artículo
function decreaseQuantity(index) {
    if (cartItems.value[index].cantidad > 1) {
        cartItems.value[index].cantidad--;
        localStorage.setItem('cart', JSON.stringify(cartItems.value));
    }
}

// Calcular el total general de productos seleccionados
const totalGeneral = computed(() => {
    return cartItems.value.reduce((total, item) => total + (item.selected ? item.precio * item.cantidad : 0), 0);
});

// Eliminar un artículo del carrito
function removeItem(index) {
    cartItems.value.splice(index, 1);
    localStorage.setItem('cart', JSON.stringify(cartItems.value));
}

// Enviar pedido por WhatsApp solo de productos seleccionados
function sendWhatsAppOrder() {
    const selectedItems = cartItems.value.filter(item => item.selected);
    const message = selectedItems.map(item => `${item.nombre} - Cantidad: ${item.cantidad} Color: ${item.color}`).join('\n');

    if (message) {
        const whatsappUrl = `https://wa.me/51936081881?text=Pedido:%0A${encodeURIComponent(message)}`;
        window.open(whatsappUrl, '_blank');
    } else {
        alert("No hay productos seleccionados para enviar.");
    }
}

onMounted(() => {
    loadCartItems();
});

// Observa cambios en el carrito
watch(() => localStorage.getItem('cart'), loadCartItems);
</script>
<template>
    <div class="cart-container">
        <div class="cart-header">
            <h2>Carrito de Compras</h2>
            <RouterLink to="/productos" class="continue-shopping" title="Seguir Comprando">
                <span class="icon">←</span>
                Seguir comprando
            </RouterLink>
        </div>

        <div v-if="cartItems.length" class="cart-content">
            <div class="cart-items">
                <div v-for="(item, index) in cartItems" :key="index" class="cart-item">
                    <div class="item-selector">
                        <label class="checkbox-wrapper">
                            <input type="checkbox" v-model="item.selected" class="checkbox" />
                            <span class="checkmark"></span>
                        </label>
                    </div>

                    <div class="item-image">

                        <img :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${item.imagen}`" :alt="item.nombre" />
                        <!-- <img :src="`http://localhost:8000/${item.imagen}`" :alt="item.nombre" /> -->
                    </div>

                    <div class="item-info">
                        <h3 class="item-name">{{ item.nombre }}</h3>
                        <div class="item-properties">
                            <span v-if="item.color" class="property">
                                <span class="property-label">Color:</span>
                                {{ item.color }}
                            </span>
                            <span v-if="item.tamano" class="property">
                                <span class="property-label">Tamaño:</span>
                                {{ item.tamano }}
                            </span>
                            <span v-if="item.longitud" class="property">
                                <span class="property-label">Longitud:</span>
                                {{ item.longitud }}
                            </span>
                        </div>
                    </div>

                    <div class="item-quantity">
                        <button @click="decreaseQuantity(index)" class="quantity-btn">−</button>
                        <input type="number" v-model.number="item.cantidad"
                            @input="validateAndUpdateQuantity($event, index)" min="1" class="quantity-input" />
                        <button @click="increaseQuantity(index)" class="quantity-btn">+</button>
                    </div>

                    <div class="item-price">
                        <div class="price">
                            S/{{ isValidprecio(item.precio) ? item.precio.toFixed(2) : 'N/A' }}
                        </div>
                        <div class="total">
                            S/{{ (item.precio * item.cantidad).toFixed(2) }}
                        </div>
                    </div>

                    <button @click="removeItem(index)" class="remove-btn" title="Eliminar producto">
                        ×
                    </button>
                </div>
            </div>

            <div class="cart-summary">
                <div class="summary-row">
                    <span>Total de productos seleccionados:</span>
                    <span class="total-amount">S/{{ totalGeneral.toFixed(2) }}</span>
                </div>

                <button @click="sendWhatsAppOrder" class="whatsapp-btn">
                    <span class="whatsapp-icon">📱</span>
                    Enviar pedido por WhatsApp
                </button>
            </div>
        </div>

        <div v-else class="empty-cart">
            <div class="empty-cart-icon">🛒</div>
            <p>Tu carrito está vacío</p>
            <a href="/productos" class="start-shopping-btn">
                Comenzar a comprar
            </a>
        </div>
    </div>
</template>



<style scoped>
.cart-container {
    max-width: 1200px;
    margin: 2rem auto;
    padding: 2rem;
    background: #ffffff;
    border-radius: 1rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.cart-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
    padding-bottom: 1rem;
    border-bottom: 1px solid #edf2f7;
}

.cart-header h2 {
    font-size: 1.8rem;
    font-weight: 600;
    color: #1a202c;
    margin: 0;
}

.continue-shopping {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: #4a5568;
    text-decoration: none;
    font-weight: 500;
    transition: color 0.2s;
}

.continue-shopping:hover {
    color: #2d3748;
}

.cart-items {
    display: flex;
    flex-direction: column;
    gap: 0.8rem;
}

.cart-item {
    display: grid;
    grid-template-columns: auto auto 3fr 0.3fr 1fr auto;
    align-items: center;
    gap: 1.5rem;
    padding: 1.5rem;
    background: #f8fafc;
    border-radius: 0.75rem;
    transition: transform 0.2s;
}

.cart-item:hover {
    transform: translateY(-2px);
}

.item-image img {
    width: 80px;
    height: 80px;
    object-fit: cover;
    border-radius: 0.5rem;
}

.item-info {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.item-name {
    font-size: 1.1rem;
    font-weight: 600;
    color: #2d3748;
    margin: 0;
}

.item-properties {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
    font-size: 0.9rem;
    color: #4a5568;
}

.property {
    display: flex;
    gap: 0.25rem;
}

.property-label {
    font-weight: 500;
}

/*afffffffffffffffffff */
.item-quantity {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    background: #fff;
    padding: 0.25rem;
    border-radius: 0.5rem;
    border: 1px solid #e2e8f0;
}

.quantity-btn {
    width: 2rem;
    height: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #f7fafc;
    border: none;
    border-radius: 0.375rem;
    color: #4a5568;
    cursor: pointer;
    transition: all 0.2s;
}

.quantity-btn:hover {
    background: #edf2f7;
    color: #2d3748;
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


.item-price {
    text-align: right;
}

.price {
    color: #4a5568;
    font-size: 0.9rem;
}

.total {
    color: #2d3748;
    font-weight: 600;
    font-size: 1.1rem;
}

.remove-btn {
    width: 2rem;
    height: 2rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background: none;
    border: none;
    color: #a0aec0;
    font-size: 1.5rem;
    cursor: pointer;
    transition: color 0.2s;
}

.remove-btn:hover {
    color: #e53e3e;
}

.cart-summary {
    margin-top: 2rem;
    padding: 2rem;
    background: #f8fafc;
    border-radius: 0.75rem;
}

.summary-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
    font-size: 1.1rem;
}

.total-amount {
    font-size: 1.5rem;
    font-weight: 600;
    color: #2d3748;
}

.whatsapp-btn {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.75rem;
    padding: 1rem;
    background: #25D366;
    color: white;
    border: none;
    border-radius: 0.5rem;
    font-size: 1.1rem;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s;
}

.whatsapp-btn:hover {
    background: #128C7E;
}

.empty-cart {
    text-align: center;
    padding: 4rem 2rem;
}

.empty-cart-icon {
    font-size: 4rem;
    margin-bottom: 1rem;
    color: #a0aec0;
}

.empty-cart p {
    color: #4a5568;
    font-size: 1.2rem;
    margin-bottom: 2rem;
}

.start-shopping-btn {
    display: inline-block;
    padding: 1rem 2rem;
    background: #4299e1;
    color: white;
    text-decoration: none;
    border-radius: 0.5rem;
    font-weight: 500;
    transition: background-color 0.2s;
}

.start-shopping-btn:hover {
    background: #3182ce;
}

.checkbox-wrapper {
    position: relative;
    display: block;
    width: 1.5rem;
    height: 1.5rem;
}

.checkbox {
    position: absolute;
    opacity: 0;
    cursor: pointer;
    width: 100%;
    height: 100%;
}

.checkmark {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: #fff;
    border: 2px solid #e2e8f0;
    border-radius: 0.375rem;
    transition: all 0.2s;
}

.checkbox:checked~.checkmark {
    background-color: #4299e1;
    border-color: #4299e1;
}

.checkmark:after {
    content: "";
    position: absolute;
    display: none;
    left: 50%;
    top: 50%;
    width: 0.5rem;
    height: 0.8rem;
    border: solid white;
    border-width: 0 2px 2px 0;
    transform: translate(-50%, -60%) rotate(45deg);
}

.checkbox:checked~.checkmark:after {
    display: block;
}

@media (max-width: 768px) {
    .cart-container {
        padding: 1rem;
        margin: 1rem;
    }

    .cart-item {
        grid-template-columns: auto 1fr auto;
        grid-template-areas:
            "selector image remove"
            "selector info info"
            "selector quantity price";
        padding: 1rem;
    }

    .item-selector {
        grid-area: selector;
    }

    .item-image {
        grid-area: image;
    }

    .item-info {
        grid-area: info;
        margin: 1rem 0;
    }

    .item-quantity {
        grid-area: quantity;
    }

    .item-price {
        grid-area: price;
        text-align: right;
    }

    .remove-btn {
        grid-area: remove;

        width: 4.3rem;
        height: 2rem;
        display: flex;
        align-items: center;
        justify-content: center;
        background: none;
        border: none;
        color: #a0aec0;
        font-size: 1.5rem;
        cursor: pointer;
        transition: color 0.2s;
    }

    .cart-header {
        flex-direction: column;
        gap: 1rem;
        align-items: flex-start;
    }

    .summary-row {
        flex-direction: column;
        gap: 0.5rem;
        text-align: center;
    }
}
</style>