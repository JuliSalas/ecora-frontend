<script setup>
import { ref, watch, onMounted } from 'vue'
const props = defineProps({
    product: {
        type: Object,
        required: true,
    },
})

const emit = defineEmits(['close', 'add-to-cart'])
const tallaSeleccionada = ref(null)
const cantidad = ref(1)
const mensajeExito = ref('')
const mensajeError = ref('')

onMounted(() => {
    if (props.product?.tallas?.length === 1 &&
        props.product.tallas[0] === 'Unique') {
        tallaSeleccionada.value = 'Unique'
    }
})

const seleccionarTalla = (talla) => {
    tallaSeleccionada.value = talla
    mensajeError.value = ''
}

const agregarAlCarrito = () => {
    mensajeError.value = ''
    mensajeExito.value = ''

    if (!tallaSeleccionada.value) {
        mensajeError.value = 'Por favor seleccioná una talla.'
        return
    }

    if (!cantidad.value || cantidad.value < 1) {
        mensajeError.value = 'La cantidad debe ser al menos 1.'
        return
    }

    emit('add-to-cart', {
        id: props.product.id,
        nombre: props.product.nombre,
        precio: props.product.precio,
        imagen: props.product.imagen,
        talla: tallaSeleccionada.value,
        cantidad: cantidad.value,
    })

    mensajeExito.value = `¡${props.product.nombre} agregado al carrito!`

    // Cerrar modal después de mostrar el mensaje
    setTimeout(() => {
        emit('close')
    }, 1800)
}
</script>

<template>
    <div class="detail-overlay" @click.self="emit('close')">
        <div class="detail-modal">
            <button class="detail-close" type="button" @click="emit('close')" aria-label="Cerrar">
                ✕
            </button>

            <div class="detail-grid">
                <img class="detail-image" :src="product?.imagen" :alt="product?.nombre" />

                <div class="detail-content">
                    <p class="detail-eyebrow">Detalle del producto</p>
                    <h2>{{ product?.nombre || 'Nombre del producto' }}</h2>
                    <p class="detail-price">{{ product?.precio || 'Precio' }}</p>

                    <section class="detail-section">
                        <h3>Descripción</h3>
                        <p>{{ product?.descripcion || 'Descripción.' }}</p>
                    </section>

                    <section class="detail-section">
                        <h3>Categoría</h3>
                        <p>{{ product?.categoria?.join(' • ') || 'Categoría.' }}</p>
                    </section>

                    <!-- TALLAS -->
                    <section class="detail-section">
                        <h3>Seleccioná una talla</h3>
                        <div class="talla-grid">
                            <button v-for="talla in product?.tallas" :key="talla" class="talla-btn"
                                :class="{ 'talla-btn--active': tallaSeleccionada === talla }" type="button"
                                @click="seleccionarTalla(talla)">
                                {{ talla }}
                            </button>
                        </div>
                    </section>

                    <!-- CANTIDAD Y BOTÓN -->
                    <section class="detail-section cart-section">
                        <div class="cart-controls">
                            <div class="quantity-wrap">
                                <button type="button" class="quantity-btn"
                                    @click="cantidad > 1 ? cantidad-- : null">−</button>
                                <input v-model.number="cantidad" class="cart-quantity" type="number" min="1" step="1">
                                <button type="button" class="quantity-btn" @click="cantidad++">+</button>
                            </div>
                            <button class="cart-button" type="button" @click="agregarAlCarrito">
                                Añadir al carrito
                            </button>
                        </div>
                        <!-- Mensajes -->
                        <p v-if="mensajeError" class="msg msg--error">{{ mensajeError }}</p>
                        <p v-if="mensajeExito" class="msg msg--success">{{ mensajeExito }}</p>
                    </section>
                </div>
            </div>
        </div>
    </div>
</template>
<style scoped>
.detail-overlay {
    position: fixed;
    inset: 0;
    background: rgba(11, 11, 9, 0.7);
    backdrop-filter: blur(8px);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1100;
    padding: 1.5rem;
}

.detail-modal {
    background: linear-gradient(145deg, #fdfbf7 0%, #f4efe7 100%);
    width: min(92vw, 900px);
    max-height: 88vh;
    overflow-y: auto;
    border-radius: 24px;
    padding: 2rem;
    position: relative;
    box-shadow: 0 24px 70px rgba(15, 15, 12, 0.25);
}

.detail-close {
    position: absolute;
    top: 1rem;
    right: 1rem;
    width: 2.5rem;
    height: 2.5rem;
    border: none;
    border-radius: 50%;
    background: rgba(113, 108, 89, 0.12);
    color: var(--ecora-black, #0B0B09);
    cursor: pointer;
}

.detail-grid {
    display: grid;
    grid-template-columns: minmax(220px, 320px) 1fr;
    gap: 1.5rem;
    align-items: start;
}

.detail-image {
    width: 100%;
    aspect-ratio: 4 / 5;
    object-fit: cover;
    border-radius: 18px;
}

.detail-content {
    display: flex;
    flex-direction: column;
    gap: 0.9rem;
}

.detail-eyebrow {
    font-family: 'Jost', sans-serif;
    font-size: 0.76rem;
    letter-spacing: 0.3em;
    text-transform: uppercase;
    color: var(--ecora-stone, #716C59);
}

.detail-content h2 {
    font-family: 'Cormorant Garamond', serif;
    font-size: clamp(1.6rem, 2.4vw, 2.2rem);
    color: var(--ecora-black, #0B0B09);
}

.detail-price {
    font-family: 'Jost', sans-serif;
    font-size: 1rem;
    font-weight: 600;
    color: var(--ecora-warm, #6F5F48);
}

.detail-section {
    padding-top: 0.4rem;
    border-top: 1px solid rgba(113, 108, 89, 0.18);
}

.detail-section h3 {
    font-family: 'Jost', sans-serif;
    font-size: 0.95rem;
    margin-bottom: 0.3rem;
    color: var(--ecora-black, #0B0B09);
}

.detail-section p {
    font-family: 'Jost', sans-serif;
    line-height: 1.6;
    color: var(--ecora-stone, #716C59);
}

.tallas-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    margin-top: 0.5rem;
}

.talla-btn {
    width: 44px;
    height: 44px;
    border: 1px solid var(--ecora-sand);
    background: var(--ecora-cream);
    font-family: 'Jost', sans-serif;
    font-size: 13px;
    cursor: pointer;
    transition: all 0.2s;
    color: var(--ecora-black);
}

.talla-btn:hover {
    border-color: var(--ecora-stone);
    background: var(--ecora-cream);
}

.talla-btn--active {
    background: #F5F3EE;
    border-color: var(--ecora-black);
    border-width: 2px;
    color: var(--ecora-black);
    font-weight: 600;
}

.cart-controls {
    display: flex;
    gap: 0.75rem;
    align-items: center;
    flex-wrap: wrap;
}
.quantity-wrap {
    display: flex;
    align-items: center;
    border: 1px solid rgba(113, 108, 89, 0.25);
    border-radius: 999px;
    background: rgba(255, 255, 255, 0.85);
    overflow: hidden;
    min-width: 110px;
}

.quantity-btn {
    width: 36px;
    height: 40px;
    background: none;
    border: none;
    font-size: 18px;
    color: var(--ecora-black);
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.15s;
    flex-shrink: 0;
}

.quantity-btn:hover {
    background: rgba(113, 108, 89, 0.1);
}

.cart-quantity {
    width: 38px;
    min-width: 38px;
    text-align: center;
    border: none;
    background: transparent;
    font-family: 'Jost', sans-serif;
    font-size: 0.95rem;
    color: var(--ecora-black);
    outline: none;
    -webkit-appearance: none;
    appearance: none;
    padding: 0;
}

.cart-quantity::-webkit-inner-spin-button,
.cart-quantity::-webkit-outer-spin-button {
    -webkit-appearance: none;
    appearance: none;
}

.cart-quantity:focus {
    border-color: var(--ecora-warm, #6F5F48);
    box-shadow: 0 0 0 3px rgba(111, 95, 72, 0.12);
}


.cart-button {
    border: none;
    border-radius: 999px;
    padding: 0.75rem 1rem;
    background: var(--ecora-black, #0B0B09);
    color: var(--ecora-cream, #F5F3EE);
    font-family: 'Jost', sans-serif;
    font-weight: 500;
    cursor: pointer;
    transition: transform 0.2s ease, background 0.2s ease;
}

.cart-button:hover {
    transform: translateY(-1px);
    background: var(--ecora-warm, #6F5F48);
}

.msg {
    font-family: 'Jost', sans-serif;
    font-size: 13px;
    margin-top: 0.75rem;
}

.msg--error {
    color: #c0392b;
}

.msg--success {
    color: #27ae60;
}

@media (max-width: 720px) {
    .detail-grid {
        grid-template-columns: 1fr;
    }

    .detail-modal {
        padding: 1.5rem;
    }

    .cart-quantity {
        width: 6rem;
        /* más ancho para que quepan las flechas */
        border-radius: 8px;
        -webkit-appearance: auto;
        appearance: auto;
    }

    .cart-quantity::-webkit-inner-spin-button,
    .cart-quantity::-webkit-outer-spin-button {
        opacity: 1;
        height: auto;
    }

    .cart-controls {
        flex-wrap: nowrap;
        /* evita que el botón se rompa a otra línea */
    }
}
</style>