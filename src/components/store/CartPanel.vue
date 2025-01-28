<script setup>
import { ref, computed, onMounted, watch } from 'vue';

// Definir la función emit para emitir eventos
const emit = defineEmits(['close', 'updateCart']);
const cartItems = ref([]);

// Computed property para verificar si hay items seleccionados
const hasSelectedItems = computed(() => {
    return cartItems.value.some(item => item.selected);
});

// Cargar los artículos del carrito desde localStorage
function loadCartItems() {
    cartItems.value = JSON.parse(localStorage.getItem('cart') || '[]');
}

// Cerrar el panel del carrito
function emitClose() {
    emit('close');
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
    else if (value >= cartItems.value[index].stock) {
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
    updateCart();
}

// Disminuir la cantidad de un artículo
function decreaseQuantity(index) {
    if (cartItems.value[index].cantidad > 1) {
        cartItems.value[index].cantidad--;
        updateCart();
    }
}

// Calcular el total general de productos seleccionados
const totalGeneral = computed(() => {
    return cartItems.value.reduce((total, item) =>
        total + (item.selected ? item.precio * item.cantidad : 0), 0);
});

// Eliminar un artículo del carrito
function removeItem(index) {
    cartItems.value.splice(index, 1);
    updateCart();
}

function updateCart() {
    localStorage.setItem('cart', JSON.stringify(cartItems.value));
    emit('updateCart');
}

// Enviar pedido por WhatsApp solo de productos seleccionados
function sendWhatsAppOrder() {
    const selectedItems = cartItems.value.filter(item => item.selected);
    const message = selectedItems.map(item => `${item.nombre} - Cantidad: ${item.quantity} Color: ${item.color}`).join('\n');

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
    <div class="cart-panel-overlay" @click="emitClose">
        <div class="cart-panel" @click.stop>
            <div class="cart-header">
                <div class="cart-title">
                    <h3>Tu Carrito</h3>
                    <span class="item-count">{{ cartItems.length }} items</span>
                </div>
                <button class="close-btn" @click="emitClose" aria-label="Cerrar carrito">
                    ×
                </button>
            </div>

            <div class="cart-content">
                <div v-if="cartItems.length" class="cart-items-container">
                    <div v-for="(item, index) in cartItems" :key="index" class="cart-item">
                        <div class="item-selection">
                            <label class="checkbox-container">
                                <input type="checkbox" v-model="item.selected" class="checkbox" />
                                <span class="checkmark"></span>
                            </label>
                        </div>

                        <div class="item-main">

                            <img :src="`https://8nkxdpw3-8000.brs.devtunnels.ms/${item.imagen}`" :alt="item.nombre"
                                class="item-image" />

                            <!-- <img :src="`http://localhost:8000/${item.imagen}`" :alt="item.nombre" class="item-image" /> -->

                            <div class="item-details">
                                <h4 class="item-name">{{ item.nombre }}</h4>
                                <div class="item-properties">
                                    <span v-if="item.color" class="property">
                                        Color: {{ item.color }}
                                    </span>
                                    <span v-if="item.tamano" class="property">
                                        Tamaño: {{ item.tamano }}
                                    </span>
                                    <span v-if="item.longitud" class="property">
                                        Longitud: {{ item.longitud }}
                                    </span>
                                </div>
                            </div>
                        </div>

                        <div class="item-controls">
                            <div class="price-info">
                                <span class="price">S/{{ isValidprecio(item.precio) ? item.precio.toFixed(2) : 'N/A'
                                    }}</span>
                                <span class="total">Total: S/{{ (item.precio * item.cantidad).toFixed(2) }}</span>
                            </div>

                            <div class="quantity-control">
                                <button @click="decreaseQuantity(index)" class="quantity-btn">−</button>
                                <input type="number" v-model.number="item.cantidad"
                                    @input="validateAndUpdateQuantity($event, index)" min="1" class="quantity-input" />
                                <button @click="increaseQuantity(index)" class="quantity-btn">+</button>
                            </div>

                            <button @click="removeItem(index)" class="remove-btn" title="Eliminar producto">
                                <span class="remove-icon">🗑</span>
                            </button>
                        </div>
                    </div>
                </div>

                <div v-else class="empty-cart">
                    <div class="empty-cart-icon">🛒</div>
                    <p>Tu carrito está vacío</p>
                    <RouterLink to="/productos" class="start-shopping" @click="emitClose">
                        Comenzar a comprar
                    </RouterLink>
                </div>
            </div>

            <div v-if="cartItems.length" class="cart-footer">
                <div class="cart-total">
                    <span>Total a pagar:</span>
                    <span class="total-amount">S/{{ totalGeneral.toFixed(2) }}</span>
                </div>

                <div class="cart-actions">
                    <RouterLink to="/carrito" class="view-cart-btn" @click="emitClose">
                        Ver carrito completo
                    </RouterLink>
                    <button class="whatsapp-btn" @click="sendWhatsAppOrder" :disabled="!hasSelectedItems">
                        <span class="whatsapp-icon">📱</span>
                        Enviar por WhatsApp
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.cart-panel-overlay {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 1000;
    display: flex;
    justify-content: flex-end;
}

.cart-panel {
    width: 400px;
    height: 100%;
    background: white;
    display: flex;
    flex-direction: column;
    box-shadow: -2px 0 8px rgba(0, 0, 0, 0.1);
    animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
    from {
        transform: translateX(100%);
    }

    to {
        transform: translateX(0);
    }
}

.cart-header {
    padding: 1.5rem;
    border-bottom: 1px solid #e2e8f0;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.cart-title {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
}

.cart-title h3 {
    margin: 0;
    font-size: 1.25rem;
    font-weight: 600;
    color: #1a202c;
}

.item-count {
    font-size: 0.875rem;
    color: #718096;
}

.close-btn {
    background: none;
    border: none;
    font-size: 1.5rem;
    color: #718096;
    cursor: pointer;
    padding: 0.5rem;
    border-radius: 0.375rem;
    transition: all 0.2s;
}

.close-btn:hover {
    color: #1a202c;
    background: #f7fafc;
}

.cart-content {
    flex: 1;
    overflow-y: auto;
    padding: 1rem;
}

.cart-items-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.cart-item {
    background: #f8fafc;
    border-radius: 0.75rem;
    padding: 1rem;
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 1rem;
}

.item-selection {
    align-self: center;
}

.item-main {
    display: flex;
    gap: 1rem;
}

.item-image {
    width: 80px;
    height: 80px;
    object-fit: cover;
    border-radius: 0.5rem;
}

.item-details {
    flex: 1;
    min-width: 0;
}

.item-name {
    margin: 0 0 0.5rem 0;
    font-size: 1rem;
    font-weight: 500;
    color: #2d3748;
}

.item-properties {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    font-size: 0.875rem;
    color: #718096;
}

.property {
    padding: 0.25rem 0.5rem;
    background: #edf2f7;
    border-radius: 0.25rem;
}

.item-controls {
    grid-column: 1 / -1;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-top: 0.5rem;
    border-top: 1px solid #e2e8f0;
    margin-top: 0.5rem;
}

.price-info {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
}

.price {
    font-size: 0.875rem;
    color: #718096;
}

.total {
    font-weight: 600;
    color: #2d3748;
}

.quantity-control {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    background: white;
    border-radius: 0.375rem;
    padding: 0.25rem;
    border: 1px solid #e2e8f0;
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

.quantity-value {
    width: 2rem;
    text-align: center;
    font-weight: 500;
}

.quantity-btn {
    width: 1.75rem;
    height: 1.75rem;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #f7fafc;
    border: none;
    border-radius: 0.25rem;
    color: #4a5568;
    cursor: pointer;
    transition: all 0.2s;
}

.quantity-btn:hover {
    background: #edf2f7;
    color: #2d3748;
}

.quantity {
    width: 1.5rem;
    text-align: center;
    font-weight: 500;
}

.remove-btn {
    background: none;
    border: none;
    padding: 0.5rem;
    color: #a0aec0;
    cursor: pointer;
    transition: color 0.2s;
}

.remove-btn:hover {
    color: #e53e3e;
}

.empty-cart {
    text-align: center;
    padding: 3rem 1rem;
}

.empty-cart-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
    color: #a0aec0;
}

.start-shopping {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background: #4299e1;
    color: white;
    text-decoration: none;
    border-radius: 0.375rem;
    font-weight: 500;
    margin-top: 1rem;
    transition: background-color 0.2s;
}

.start-shopping:hover {
    background: #3182ce;
}

.cart-footer {
    border-top: 1px solid #e2e8f0;
    padding: 1.5rem;
    background: white;
}

.cart-total {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
    font-size: 1.125rem;
}

.total-amount {
    font-weight: 600;
    color: #2d3748;
}

.cart-actions {
    display: flex;
    gap: 1rem;
}

.view-cart-btn {
    flex: 1;
    padding: 0.75rem;
    background: #f7fafc;
    color: #2d3748;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    text-decoration: none;
    text-align: center;
    transition: all 0.2s;
}

.view-cart-btn:hover {
    background: #edf2f7;
}

.whatsapp-btn {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    padding: 0.75rem;
    background: #25D366;
    color: white;
    border: none;
    border-radius: 0.375rem;
    cursor: pointer;
    transition: background-color 0.2s;
}

.whatsapp-btn:not(:disabled):hover {
    background: #128C7E;
}

.whatsapp-btn:disabled {
    background: #A0AEC0;
    cursor: not-allowed;
}

.checkbox-container {
    position: relative;
    display: block;
    width: 1.25rem;
    height: 1.25rem;
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
    background: white;
    border: 2px solid #e2e8f0;
    border-radius: 0.25rem;
    transition: all 0.2s;
}

.checkbox:checked~.checkmark {
    background: #4299e1;
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
</style>