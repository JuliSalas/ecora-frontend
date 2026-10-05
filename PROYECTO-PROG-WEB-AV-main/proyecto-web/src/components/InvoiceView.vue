<script setup>
import { ref, computed } from 'vue'
import html2pdf from 'html2pdf.js'

const props = defineProps({
  usuario: {
    type: Object,
    required: true
    // Ej: { nombre: 'María Jiménez', email: 'maria@correo.com', telefono: '8888-8888' }
  },
  items: {
    type: Array,
    required: true
    // Ej: [{ id: 1, nombre: 'Camiseta Manifiesto', talla: 'M', cantidad: 2, precio: 25000 }]
  },
  numeroOrden: {
    type: [String, Number],
    required: true
  },
  fecha: {
    type: [String, Date],
    default: () => new Date()
  },
  total: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['close'])

const invoiceRef = ref(null)
const descargando = ref(false)
const errorDescarga = ref(false)

const formatearPrecio = (valor) => {
  const numero = parseInt(`${valor}`.replace(/\D/g, ''))
  return `CRC ${numero.toLocaleString('es-CR')}`
}

const precioUnitario = (precio) => {
  return parseInt(`${precio}`.replace(/\D/g, ''))
}

const subtotal = computed(() => {
  return props.items.reduce(
    (acc, item) => acc + precioUnitario(item.precio) * item.cantidad,
    0
  )
})

const impuesto = computed(() => {
  const diferencia = props.total - subtotal.value
  return diferencia > 0 ? diferencia : 0
})

const fechaFormateada = computed(() => {
  const d = new Date(props.fecha)
  return d.toLocaleDateString('es-CR', {
    day: 'numeric',
    month: 'long',
    year: 'numeric'
  })
})

const numeroOrdenFormateado = computed(() => {
  return `${props.numeroOrden}`.padStart(6, '0')
})

const descargarPDF = async () => {
  if (!invoiceRef.value || descargando.value) return

  descargando.value = true
  errorDescarga.value = false

  const opciones = {
    margin: 0,
    filename: `factura-ecora-${numeroOrdenFormateado.value}.pdf`,
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2, useCORS: true, backgroundColor: '#ffffff' },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  }

  try {
    await html2pdf().set(opciones).from(invoiceRef.value).save()
  } catch (e) {
    console.error('Error generando el PDF:', e)
    errorDescarga.value = true
  } finally {
    descargando.value = false
  }
}

defineExpose({ descargarPDF })
</script>

<template>
  <div class="invoice-screen">
    <!-- Acciones (no se incluyen en el PDF) -->
    <div class="invoice-actions">
      <button class="invoice-btn-volver" type="button" @click="emit('close')">
        ← Volver a la tienda
      </button>

      <div class="invoice-actions__right">
        <span v-if="errorDescarga" class="invoice-error">
          No se pudo generar el PDF. Intentá de nuevo.
        </span>
        <button
          class="invoice-btn-descargar"
          type="button"
          :disabled="descargando"
          @click="descargarPDF"
        >
          {{ descargando ? 'Generando PDF…' : 'Descargar factura PDF' }}
        </button>
      </div>
    </div>

    <!-- Contenido que se convierte en PDF -->
    <div ref="invoiceRef" class="invoice">
      <!-- Header -->
      <header class="invoice-header">
        <div class="invoice-brand">
          <img src="/favicon.png" alt="Ecora" class="invoice-brand__logo">
          <span class="invoice-brand__name">ECORA</span>
          <span class="invoice-brand__sub">MODA SOSTENIBLE</span>
        </div>

        <div class="invoice-meta">
          <p class="invoice-meta__numero">FACTURA N.º {{ numeroOrdenFormateado }}</p>
          <p class="invoice-meta__fecha">Fecha: {{ fechaFormateada }}</p>
        </div>
      </header>

      <!-- Datos del cliente -->
      <section class="invoice-client">
        <h2 class="invoice-client__nombre">{{ usuario.nombre }}</h2>
        <div class="invoice-client__divider"></div>
        <p v-if="usuario.email" class="invoice-client__dato">{{ usuario.email }}</p>
        <p v-if="usuario.telefono" class="invoice-client__dato">{{ usuario.telefono }}</p>
        <p v-if="usuario.direccion" class="invoice-client__dato">{{ usuario.direccion }}</p>
      </section>

      <!-- Tabla de productos -->
      <div class="invoice-table-wrap">
        <table class="invoice-table">
          <thead>
            <tr>
              <th class="col-concepto">Concepto</th>
              <th class="col-talla">Talla</th>
              <th class="col-cantidad">Cantidad</th>
              <th class="col-precio">Precio</th>
              <th class="col-total">Total</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in items" :key="`${item.id}-${item.talla}-${index}`">
              <td class="col-concepto">{{ item.nombre }}</td>
              <td class="col-talla">{{ item.talla }}</td>
              <td class="col-cantidad">{{ item.cantidad }}</td>
              <td class="col-precio">{{ formatearPrecio(item.precio) }}</td>
              <td class="col-total">
                {{ formatearPrecio(precioUnitario(item.precio) * item.cantidad) }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Resumen -->
      <section class="invoice-summary">
        <div class="invoice-summary__row">
          <span>Subtotal</span>
          <span>{{ formatearPrecio(subtotal) }}</span>
        </div>
        <div v-if="impuesto > 0" class="invoice-summary__row">
          <span>IVA</span>
          <span>{{ formatearPrecio(impuesto) }}</span>
        </div>
        <div class="invoice-summary__row invoice-summary__row--total">
          <span>Total</span>
          <span>{{ formatearPrecio(total) }}</span>
        </div>
      </section>

      <!-- Footer -->
      <footer class="invoice-footer">
        <p>WWW.ECORA.COM</p>
      </footer>

      <div class="invoice-bar"></div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500&family=Jost:wght@300;400;500;600&display=swap');

.invoice-screen {
  position: fixed;
  inset: 0;
  z-index: 500;
  overflow-y: auto;
  min-height: 100vh;
  background: #E7E4DA;
  padding: 2.5rem 1.5rem 4rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

/* ── Acciones ── */
.invoice-actions {
  width: 100%;
  max-width: 900px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  flex-wrap: wrap;
}

.invoice-actions__right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.invoice-btn-volver {
  background: none;
  border: none;
  font-family: 'Jost', sans-serif;
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.05em;
  color: var(--ecora-stone);
  cursor: pointer;
  padding: 0.5rem 0;
  transition: color 0.2s;
}

.invoice-btn-volver:hover {
  color: var(--ecora-black);
}

.invoice-error {
  font-size: 12px;
  color: var(--ecora-error);
}

.invoice-btn-descargar {
  padding: 0.875rem 1.75rem;
  background: var(--ecora-stone);
  color: var(--ecora-cream);
  border: none;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
  transition: background 0.2s;
  white-space: nowrap;
}

.invoice-btn-descargar:hover:not(:disabled) {
  background: var(--ecora-black);
}

.invoice-btn-descargar:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ── Documento ── */
.invoice {
  width: 100%;
  max-width: 900px;
  background: var(--ecora-cream);
  padding: 3rem 3rem 0;
  position: relative;
  box-shadow: 0 4px 24px rgba(11, 11, 9, 0.08);
}

/* ── Header ── */
.invoice-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 1.5rem;
  margin-bottom: 3rem;
}

.invoice-brand {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  background: var(--ecora-warm);
  padding: 1.75rem 2.25rem 1.5rem;
  border-radius: 20px 20px 999px 999px;
  min-width: 160px;
}

.invoice-brand__logo {
  width: 36px;
  height: 36px;
  object-fit: contain;
  filter: brightness(0) invert(1);
}

.invoice-brand__name {
  font-family: 'Cormorant Garamond', serif;
  font-size: 20px;
  font-weight: 400;
  letter-spacing: 0.25em;
  color: var(--ecora-cream);
}

.invoice-brand__sub {
  font-family: 'Jost', sans-serif;
  font-size: 8px;
  font-weight: 500;
  letter-spacing: 0.2em;
  color: var(--ecora-sand);
}

.invoice-meta {
  text-align: right;
  padding-top: 0.5rem;
}

.invoice-meta__numero {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 0.05em;
  color: var(--ecora-black);
  margin: 0 0 0.4rem;
  word-break: break-all;
}

.invoice-meta__fecha {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  color: var(--ecora-stone);
  margin: 0;
}

/* ── Cliente ── */
.invoice-client {
  margin-bottom: 2.5rem;
}

.invoice-client__nombre {
  font-family: 'Cormorant Garamond', serif;
  font-size: 26px;
  font-weight: 400;
  color: var(--ecora-black);
  margin: 0 0 0.75rem;
}

.invoice-client__divider {
  height: 1px;
  background: var(--ecora-sand);
  opacity: 0.6;
  margin-bottom: 0.75rem;
}

.invoice-client__dato {
  font-size: 13px;
  color: var(--ecora-stone);
  margin: 0 0 0.3rem;
}

/* ── Tabla ── */
.invoice-table-wrap {
  overflow-x: auto;
}

.invoice-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 1.5rem;
  table-layout: fixed;
}

/* Anchos de columnas */
.col-concepto  { width: 35%; text-align: left; }
.col-talla     { width: 13%; text-align: center; }
.col-cantidad  { width: 12%; text-align: center; }
.col-precio    { width: 20%; text-align: right; }
.col-total     { width: 20%; text-align: right; }

.invoice-table thead th {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--ecora-black);
  white-space: nowrap;
}

.invoice-table tbody td {
  font-size: 14px;
  color: var(--ecora-black);
  padding: 0.9rem 0.5rem;
  border-bottom: 1px solid rgba(174, 168, 151, 0.4);
}

.col-talla,
.col-cantidad,
.col-precio,
.col-total {
  text-align: right;
}

.col-talla {
  text-align: center;
}

/* ── Resumen ── */
.invoice-summary {
  max-width: 280px;
  margin-left: auto;
  margin-bottom: 3rem;
}

.invoice-summary__row {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
  padding: 0.6rem 0;
  border-bottom: 1px solid rgba(174, 168, 151, 0.4);
  font-size: 14px;
  color: var(--ecora-stone);
}

.invoice-summary__row--total {
  border-bottom: 1px solid var(--ecora-black);
  font-weight: 600;
  color: var(--ecora-black);
  font-size: 16px;
}

/* ── Footer ── */
.invoice-footer {
  text-align: center;
  padding: 1.5rem 0 2.5rem;
}

.invoice-footer p {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.2em;
  color: var(--ecora-stone);
  margin: 0;
}

.invoice-bar {
  height: 20px;
  background: var(--ecora-warm);
  margin: 0 -3rem;
}

/* ── Impresión (Ctrl+P como alternativa a la descarga) ── */
@media print {
  .invoice-actions {
    display: none;
  }
  .invoice-screen {
    background: white;
    padding: 0;
  }
  .invoice {
    box-shadow: none;
  }
}

/* ── Responsive ── */
@media (max-width: 600px) {
  .invoice {
    padding: 2rem 1.25rem 0;
  }

  .invoice-bar {
    margin: 0 -1.25rem;
  }

  .invoice-header {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  .invoice-meta {
    text-align: center;
    width: 100%;
  }

  .invoice-meta__numero {
    font-size: 11px;
    word-break: break-all;
  }

  .invoice-brand {
    min-width: unset;
    width: 140px;
  }

  /* 👈 esto es lo importante para la tabla */
  .invoice-table-wrap {
    overflow-x: auto;
    -webkit-overflow-scrolling: touch;
    margin: 0 -1.25rem;        /* sangría negativa para aprovechar el ancho */
    padding: 0 1.25rem;
  }

  .invoice-table {
    min-width: 480px;           /* tabla siempre de 480px mínimo */
    table-layout: fixed;
  }

}
</style>