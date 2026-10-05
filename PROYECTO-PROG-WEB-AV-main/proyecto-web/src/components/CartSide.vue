<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import CartCheckout from './CartCheckout.vue'

const props = defineProps({
  items: {
    type: Array,
    required: true,
  },
  total: {
    type: Number,
    required: true,
  },
  usuario: {
    type: Object,
    default: null
  }
})

watch(() => props.usuario, (val) => {
}, { immediate: true })

const emit = defineEmits([
  'close',
  'remove-item',
  'cart-paid'
])

const confirmandoIndex = ref(null)
const checkoutOpen = ref(false)

// Cerrar con ESC
const handleKeydown = (e) => {
  if (e.key === 'Escape') emit('close')
}

onMounted(() => document.addEventListener('keydown', handleKeydown))
onUnmounted(() => document.removeEventListener('keydown', handleKeydown))

const pedirConfirmacion = (index) => {
  confirmandoIndex.value = index
}

const confirmarEliminar = (index) => {
  emit('remove-item', index)
  confirmandoIndex.value = null
}

const cancelarEliminar = () => {
  confirmandoIndex.value = null
}

const formatearPrecio = (precio) => {
  const numero = parseInt(`${precio}`.replace(/\D/g, ''))

  return `CRC ${numero.toLocaleString('es-CR')}`
}

const precioUnitario = (precio) => {
  return parseInt(`${precio}`.replace(/\D/g, ''))
}

const completarCompra = () => {
  if (!props.usuario || props.items.length === 0) {
    return
  }

  const nuevaOrden = {
    cliente: {
      id: props.usuario.id,
      nombre: props.usuario.nombre,
      correo: props.usuario.correo
    },

    productos: props.items.map((item) => ({
      id: item.id,
      nombre: item.nombre,
      talla: item.talla,
      cantidad: Number(item.cantidad),
      precio: precioUnitario(item.precio),
      imagen: item.imagen
    })),

    total: Number(props.total),
    fecha: new Date().toISOString(),
    estado: 'Pendiente'
  }

  checkoutOpen.value = false
  emit('cart-paid', nuevaOrden)
}
</script>

<template>
  <!-- Overlay -->
  <div
    class="cart-overlay"
    @click="emit('close')"
  ></div>

  <!-- Panel lateral -->
  <aside class="cart-side">

    <!-- Header -->
    <div class="cart-header">
      <div class="cart-header__title">
        <svg
          width="20"
          height="20"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="1.5"
        >
          <path
            d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z"
          />

          <line
            x1="3"
            y1="6"
            x2="21"
            y2="6"
          />

          <path
            d="M16 10a4 4 0 01-8 0"
          />
        </svg>

        <span>
          Tu bolsa ({{ items.length }})
        </span>
      </div>

      <button
        class="cart-header__close"
        aria-label="Cerrar carrito"
        @click="emit('close')"
      >
        ✕
      </button>
    </div>

    <div class="cart-divider"></div>

    <!-- Lista de items -->
    <div class="cart-body">

      <!-- Vacío -->
      <div
        v-if="items.length === 0"
        class="cart-empty"
      >
        <svg
          width="48"
          height="48"
          viewBox="0 0 24 24"
          fill="none"
          stroke="#AEA897"
          stroke-width="1"
        >
          <path
            d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z"
          />

          <line
            x1="3"
            y1="6"
            x2="21"
            y2="6"
          />

          <path
            d="M16 10a4 4 0 01-8 0"
          />
        </svg>

        <p class="cart-empty__text">
          Tu bolsa está vacía.
        </p>

        <p class="cart-empty__sub">
          Explora la colección.
        </p>
      </div>

      <!-- Items -->
      <template v-else>
        <article
          v-for="(item, index) in items"
          :key="`${item.id}-${item.talla}-${index}`"
          class="cart-item"
        >
          <!-- Imagen -->
          <img
            :src="item.imagen"
            :alt="item.nombre"
            class="cart-item__img"
          >

          <!-- Info -->
          <div class="cart-item__info">
            <h4 class="cart-item__nombre">
              {{ item.nombre }}
            </h4>

            <p class="cart-item__meta">
              Talla: {{ item.talla }}
            </p>

            <p class="cart-item__meta">
              Cantidad: {{ item.cantidad }}
            </p>

            <p class="cart-item__precio-unit">
              {{ formatearPrecio(item.precio) }} c/u
            </p>

            <p class="cart-item__precio-total">
              {{
                formatearPrecio(
                  precioUnitario(item.precio) *
                  item.cantidad
                )
              }}
            </p>
          </div>

          <!-- Botón eliminar -->
          <button
            v-if="confirmandoIndex !== index"
            class="cart-item__remove"
            aria-label="Eliminar producto"
            @click="pedirConfirmacion(index)"
          >
            ✕
          </button>

          <!-- Confirmación -->
          <div
            v-else
            class="cart-item__confirm"
          >
            <p>¿Eliminar?</p>

            <div class="cart-item__confirm-btns">
              <button
                class="btn-si"
                @click="confirmarEliminar(index)"
              >
                Sí
              </button>

              <button
                class="btn-no"
                @click="cancelarEliminar"
              >
                No
              </button>
            </div>
          </div>
        </article>
      </template>
    </div>

    <!-- Footer con total y botón pagar -->
    <div
      v-if="items.length > 0"
      class="cart-footer"
    >
      <div class="cart-footer__total">
        <span class="cart-footer__total-label">
          Total a pagar
        </span>

        <span class="cart-footer__total-valor">
          CRC {{ total.toLocaleString('es-CR') }}
        </span>
      </div>

      <!-- Usuario logueado → botón pagar -->
      <button
        v-if="usuario"
        class="cart-footer__btn"
        @click="checkoutOpen = true"
      >
        Pagar — CRC {{ total.toLocaleString('es-CR') }}
      </button>

      <!-- Sin sesión → mensaje -->
      <div
        v-else
        class="cart-login-msg"
      >
        <p>
          Debés iniciar sesión para continuar con el pago.
        </p>
      </div>
    </div>
  </aside>

  <!-- Checkout -->
  <CartCheckout
  v-if="checkoutOpen"
  :total="total"
  :items="items"
  :usuario="usuario"
  @close="checkoutOpen = false"
  @paid="(orden) => emit('cart-paid', orden)"
  />
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400&family=Jost:wght@300;400;500&display=swap');

/* ── Overlay ── */
.cart-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  z-index: 200;
}

/* ── Panel ── */
.cart-side {
  position: fixed;
  top: 0;
  right: 0;
  width: 420px;
  max-width: 100vw;
  height: 100dvh;
  background: #F5F3EE;
  z-index: 201;
  display: flex;
  flex-direction: column;
  box-shadow: -4px 0 24px rgba(0, 0, 0, 0.12);
}

/* ── Header ── */
.cart-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.25rem 1.5rem;
}

.cart-header__title {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  font-weight: 500;
  color: #0B0B09;
}

.cart-header__close {
  background: none;
  border: none;
  font-size: 18px;
  color: #716C59;
  cursor: pointer;
  padding: 4px;
  transition: color 0.2s;
}

.cart-header__close:hover {
  color: #0B0B09;
}

.cart-divider {
  height: 1px;
  background: #AEA897;
  opacity: 0.3;
}

/* ── Body ── */
.cart-body {
  flex: 1;
  overflow-y: auto;
  padding: 1.25rem 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

/* ── Vacío ── */
.cart-empty {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  padding: 3rem 0;
}

.cart-empty__text {
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  color: #0B0B09;
}

.cart-empty__sub {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  color: #AEA897;
}

/* ── Item ── */
.cart-item {
  display: flex;
  gap: 12px;
  padding-bottom: 1.25rem;
  border-bottom: 1px solid rgba(174, 168, 151, 0.3);
  align-items: flex-start;
  width: 100%;
}

.cart-item__img {
  width: 70px;
  height: 70px;
  object-fit: cover;
  display: block;
  flex-shrink: 0;
}

.cart-item__info {
  flex: 1;
  min-width: 0;
}

.cart-item__nombre {
  font-family: 'Cormorant Garamond', serif;
  font-size: 16px;
  font-weight: 400;
  color: #0B0B09;
  margin: 0;
  word-break: break-word;
  overflow-wrap: break-word;
}

.cart-item-info h4 {
  word-break: break-word;
}

.cart-item__meta {
  font-family: 'Jost', sans-serif;
  font-size: 12px;
  color: #716C59;
}

.cart-item__precio-unit {
  font-family: 'Jost', sans-serif;
  font-size: 12px;
  color: #AEA897;
  margin-top: 0.25rem;
}

.cart-item__precio-total {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #0B0B09;
}

/* ── Botón eliminar ── */
.cart-item__remove {
  background: none;
  border: none;
  color: #AEA897;
  font-size: 14px;
  cursor: pointer;
  padding: 2px;
  transition: color 0.2s;
  line-height: 1;
  flex-shrink: 0;
}

.cart-item__remove:hover {
  color: #c0392b;
}

/* ── Confirmación ── */
.cart-item__confirm {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.4rem;
}

.cart-item__confirm p {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  color: #716C59;
  white-space: normal;
  text-align: center;
}

.cart-item__confirm-btns {
  display: flex;
  gap: 0.4rem;
}

.btn-si,
.btn-no {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  padding: 0.3rem 0.75rem;
  cursor: pointer;
  border: 1px solid;
  transition: all 0.15s;
}

.btn-si {
  background: #c0392b;
  color: white;
  border-color: #c0392b;
}

.btn-si:hover {
  background: #a93226;
}

.btn-no {
  background: transparent;
  color: #716C59;
  border-color: #AEA897;
}

.btn-no:hover {
  background: #F5F3EE;
  border-color: #716C59;
}

/* ── Footer ── */
.cart-footer {
  padding: 1.25rem 1.5rem;
  border-top: 1px solid rgba(174, 168, 151, 0.3);
  display: flex;
  flex-direction: column;
  gap: 1rem;
  flex-shrink: 0;
}

.cart-footer__total {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.cart-footer__total-label {
  flex-shrink: 0;
}

.cart-footer__total-valor {
  font-size: 20px;
  text-align: right;
  min-width: 0;
}

.cart-footer__total-label {
  font-family: 'Jost', sans-serif;
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #716C59;
}

.cart-footer__btn {
  width: 100%;
  padding: 1rem;
  background: #716C59;
  color: #F5F3EE;
  border: none;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
  transition: background 0.2s;
}

.cart-footer__btn:hover {
  background: #0B0B09;
}

.cart-login-msg {
  padding: 0.875rem 1rem;
  background: rgba(174, 168, 151, 0.15);
  border: 1px solid #AEA897;
  text-align: center;
}

.cart-login-msg p {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  color: #716C59;
}

/* ── Responsive ── */
@media (max-width: 480px) {

  .cart-body {
    padding: 1rem;
  }

  .cart-header {
    padding: 1rem;
  }

  .cart-footer {
    padding: 1rem;
  }

  .cart-item {
    gap: 10px;
  }

  .cart-item__img {
    width: 60px;
    height: 60px;
  }

  .cart-item__nombre {
    font-size: 15px;
  }

  .cart-item__meta {
    font-size: 11px;
  }

  .cart-footer__total {
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }

  .cart-footer__total-valor {
    font-size: 22px;
  }
}
</style>