<script setup>
import { ref, computed } from 'vue'
import InvoiceView from './InvoiceView.vue'

const props = defineProps({
  total: {
    type: Number,
    required: true,
  },
  items: {
    type: Array,
    default: () => [] },
  usuario: {
    type: Object,
    default: () => ({}) }
})

const emit = defineEmits(['close', 'paid'])

const numero = ref('')
const vencimiento = ref('')
const cvv = ref('')
const errores = ref({})
const procesando = ref(false)
const estadoPago = ref('') // 'procesando' | 'exito' | 'error'
const mostrarFactura = ref(false)
const ordenId = ref('')

// Detectar tipo de tarjeta
const tipoTarjeta = computed(() => {
  const n = numero.value.replace(/\s/g, '')
  if (/^4/.test(n)) return 'Visa'
  if (/^5[1-5]/.test(n) || /^2[2-7]/.test(n)) return 'Mastercard'
  if (/^3[47]/.test(n)) return 'American Express'
  return ''
})

const iconoTarjeta = computed(() => {
  if (tipoTarjeta.value === 'Visa') return '💳 Visa'
  if (tipoTarjeta.value === 'Mastercard') return '💳 Mastercard'
  if (tipoTarjeta.value === 'American Express') return '💳 Amex'
  return ''
})

// Formatear número de tarjeta con espacios
const formatearNumero = (e) => {
  let val = e.target.value.replace(/\D/g, '')
  // Amex: 4-6-5, resto: 4-4-4-4
  if (/^3[47]/.test(val)) {
    val = val.replace(/(\d{4})(\d{6})(\d{0,5})/, '$1 $2 $3').trim()
  } else {
    val = val.replace(/(\d{4})(?=\d)/g, '$1 ').trim()
  }
  numero.value = val
}

// Formatear vencimiento MM/AA
const formatearVencimiento = (e) => {
  let val = e.target.value.replace(/\D/g, '')
  if (val.length >= 2) {
    val = val.slice(0, 2) + '/' + val.slice(2, 4)
  }
  vencimiento.value = val
}

const validar = () => {
  const err = {}
  const n = numero.value.replace(/\s/g, '')

  if (!n) {
    err.numero = 'El número de tarjeta es requerido.'
  } else if (!tipoTarjeta.value) {
    err.numero = 'Ingresá un número de tarjeta válido (Visa, Mastercard o Amex).'
  } else if (tipoTarjeta.value === 'American Express' && n.length !== 15) {
    err.numero = 'Amex debe tener 15 dígitos.'
  } else if (tipoTarjeta.value !== 'American Express' && n.length !== 16) {
    err.numero = 'La tarjeta debe tener 16 dígitos.'
  }

  if (!vencimiento.value) {
    err.vencimiento = 'La fecha de vencimiento es requerida.'
  } else if (!/^\d{2}\/\d{2}$/.test(vencimiento.value)) {
    err.vencimiento = 'Formato inválido. Usá MM/AA.'
  } else {
    const [mes, anio] = vencimiento.value.split('/').map(Number)
    const ahora = new Date()
    const anioCompleto = 2000 + anio
    if (mes < 1 || mes > 12) {
      err.vencimiento = 'Mes inválido.'
    } else if (
      anioCompleto < ahora.getFullYear() ||
      (anioCompleto === ahora.getFullYear() && mes < ahora.getMonth() + 1)
    ) {
      err.vencimiento = 'La tarjeta está vencida.'
    }
  }

  const cvvLen = tipoTarjeta.value === 'American Express' ? 4 : 3
  if (!cvv.value) {
    err.cvv = 'El CVV es requerido.'
  } else if (cvv.value.length !== cvvLen) {
    err.cvv = `El CVV debe tener ${cvvLen} dígitos.`
  }

  errores.value = err
  return Object.keys(err).length === 0
}

const itemsFactura = ref([])
const fechaOrden = ref(null)
const ordenParaRegistrar = ref(null)

const pagar = async () => {
  if (!validar()) return

  estadoPago.value = 'procesando'

  await new Promise(resolve => setTimeout(resolve, 2000))

  const exito = Math.random() > 0.1

  if (!exito) {
    estadoPago.value = 'error'
    return
  }

  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const idsProductos = props.items.map((item) => {
      return item._id || item.id
    })

    const response = await fetch('/api/ordenes', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        productos: idsProductos,
        estado: 'Pendiente',
        total: `CRC ${props.total.toLocaleString('es-CR')}`
      })
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj || 'No fue posible registrar la orden.'
      )
    }

    console.log('ORDEN CREADA:', resultado)

    ordenId.value = resultado.orden.id
    itemsFactura.value = [...props.items]

    ordenParaRegistrar.value = {
      id: resultado.orden.id,
      fecha: resultado.orden.fecha,
      productos: [...props.items],
      total: props.total,
      cliente: {
        id: props.usuario?.id,
        nombre: props.usuario?.nombre,
        correo: props.usuario?.correo
      },
      estado: resultado.orden.estado
    }

    estadoPago.value = 'exito'

    setTimeout(() => {
      mostrarFactura.value = true
    }, 2500)

  } catch (error) {
    console.error('Error al crear la orden:', error)
    estadoPago.value = 'error'
  }
}
</script>

<template>
  <div class="checkout-overlay" @click.self="emit('close')">
    <div class="checkout-modal">

      <button class="checkout-close" @click="emit('close')" aria-label="Cerrar">✕</button>

      <!-- Estado: procesando -->
      <div v-if="estadoPago === 'procesando'" class="checkout-estado">
        <div class="checkout-spinner"></div>
        <p class="checkout-estado__texto">Su pago está siendo procesado...</p>
      </div>

      <!-- Estado: éxito -->
      <div v-else-if="estadoPago === 'exito'" class="checkout-estado">
        <span class="checkout-estado__icono">✓</span>
        <p class="checkout-estado__texto">¡Pago realizado con éxito!</p>
        <p class="checkout-estado__sub">Gracias por tu compra en Ecora.</p>
      </div>
      <!-- Estado: error -->
      <div v-else-if="estadoPago === 'error'" class="checkout-estado">
        <span class="checkout-estado__icono checkout-estado__icono--error">✕</span>
        <p class="checkout-estado__texto">No se pudo procesar el pago.</p>
        <p class="checkout-estado__sub">Verificá los datos e intentá nuevamente.</p>
        <button class="checkout-retry" @click="estadoPago = ''">Intentar de nuevo</button>
      </div>
      <InvoiceView
        v-if="mostrarFactura"
        :usuario="props.usuario"
        :items="itemsFactura"
        :numero-orden="ordenId"
        :fecha="new Date()" :total="props.total" @close="() => {
          mostrarFactura = false
          emit('paid', ordenParaRegistrar)
        }" />

      <!-- Formulario -->
      <template v-else>
        <div class="checkout-header">
          <h2 class="checkout-title">Datos de pago</h2>
          <p class="checkout-total">
            Total: CRC {{ total.toLocaleString('es-CR') }}
          </p>
        </div>

        <form class="checkout-form" @submit.prevent="pagar">

          <!-- Número de tarjeta -->
          <div class="form-group">
            <label class="form-label">
              Número de tarjeta
              <span v-if="iconoTarjeta" class="form-card-type">{{ iconoTarjeta }}</span>
            </label>
            <input
              class="form-input"
              :class="{ 'form-input--error': errores.numero }"
              type="text"
              placeholder="0000 0000 0000 0000"
              maxlength="19"
              inputmode="numeric"
              :value="numero"
              @input="formatearNumero"
            >
            <p v-if="errores.numero" class="form-error">{{ errores.numero }}</p>
          </div>

          <!-- Vencimiento y CVV -->
          <div class="form-row">
            <div class="form-group">
              <label class="form-label">Fecha de vencimiento</label>
              <input
                class="form-input"
                :class="{ 'form-input--error': errores.vencimiento }"
                type="text"
                placeholder="MM/AA"
                maxlength="5"
                inputmode="numeric"
                :value="vencimiento"
                @input="formatearVencimiento"
              >
              <p v-if="errores.vencimiento" class="form-error">{{ errores.vencimiento }}</p>
            </div>

            <div class="form-group">
              <label class="form-label">CVV</label>
              <input
                v-model="cvv"
                class="form-input"
                :class="{ 'form-input--error': errores.cvv }"
                type="password"
                :placeholder="tipoTarjeta === 'American Express' ? '0000' : '000'"
                :maxlength="tipoTarjeta === 'American Express' ? 4 : 3"
                inputmode="numeric"
              >
              <p v-if="errores.cvv" class="form-error">{{ errores.cvv }}</p>
            </div>
          </div>

          <button class="checkout-btn" type="submit">
            Confirmar pago — CRC {{ total.toLocaleString('es-CR') }}
          </button>

        </form>
      </template>

    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400&family=Jost:wght@300;400;500&display=swap');

.checkout-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 300;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.checkout-modal {
  background: #F5F3EE;
  width: 100%;
  max-width: 460px;
  padding: 2.5rem;
  position: relative;
}

.checkout-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 16px;
  color: #716C59;
  cursor: pointer;
}

/* ── Header ── */
.checkout-header {
  margin-bottom: 2rem;
}

.checkout-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: 28px;
  font-weight: 400;
  color: #0B0B09;
  margin: 0 0 0.25rem;
}

.checkout-total {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  color: #716C59;
}

/* ── Form ── */
.checkout-form {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.form-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 1fr);
  gap: 1rem;
}

.form-row .form-group {
  justify-content: flex-end;
}

.form-label {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #716C59;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.form-card-type {
  font-size: 11px;
  color: #0B0B09;
  font-weight: 400;
  letter-spacing: 0;
  text-transform: none;
}

.form-input {
  padding: 0.875rem 1rem;
  border: 1px solid #AEA897;
  background: white;
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  color: #0B0B09;
  outline: none;
  transition: border-color 0.2s;
  letter-spacing: 0.05em;
}

.form-input:focus {
  border-color: #716C59;
}

.form-input--error {
  border-color: #c0392b;
}

.form-error {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  color: #c0392b;
}

.checkout-btn {
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
  margin-top: 0.5rem;
}

.checkout-btn:hover {
  background: #0B0B09;
}

/* ── Estados ── */
.checkout-estado {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  min-height: 200px;
  text-align: center;
}

.checkout-spinner {
  width: 40px;
  height: 40px;
  border: 2px solid #AEA897;
  border-top-color: #716C59;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.checkout-estado__icono {
  font-size: 2.5rem;
  color: #27ae60;
}

.checkout-estado__icono--error {
  color: #c0392b;
}

.checkout-estado__texto {
  font-family: 'Cormorant Garamond', serif;
  font-size: 22px;
  color: #0B0B09;
}

.checkout-estado__sub {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  color: #716C59;
}

.checkout-retry {
  margin-top: 0.5rem;
  padding: 0.75rem 1.5rem;
  background: #716C59;
  color: #F5F3EE;
  border: none;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
}

</style>