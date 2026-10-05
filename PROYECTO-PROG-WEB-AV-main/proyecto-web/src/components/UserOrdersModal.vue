<script setup>
const props = defineProps({
  ordenes: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['close'])

const formatoPrecio = (precio) => {
  const numero = typeof precio === 'number'
    ? precio
    : Number(`${precio}`.replace(/\D/g, '')) || 0

  return new Intl.NumberFormat('es-CR', {
    style: 'currency',
    currency: 'CRC',
    maximumFractionDigits: 0
  }).format(numero)
}

const formatoFecha = (fecha) => {
  if (!fecha) {
    return 'Fecha no disponible'
  }

  return new Intl.DateTimeFormat('es-CR', {
    dateStyle: 'medium',
    timeStyle: 'short'
  }).format(new Date(fecha))
}

const claseEstado = (estado) => {
  return {
    'orders-status--pending': estado === 'Pendiente',
    'orders-status--confirmed': estado === 'Confirmada',
    'orders-status--preparing': estado === 'En preparación',
    'orders-status--sent': estado === 'Enviada',
    'orders-status--delivered': estado === 'Entregada',
    'orders-status--cancelled': estado === 'Cancelada'
  }
}
</script>

<template>
  <div
    class="orders-overlay"
    @click.self="emit('close')"
  >
    <section class="orders-modal">
      <div class="orders-header">
        <div>
          <p>MIS COMPRAS</p>
          <h2>Estado de mis órdenes</h2>
        </div>

        <button
          class="orders-close"
          type="button"
          aria-label="Cerrar órdenes"
          @click="emit('close')"
        >
          ×
        </button>
      </div>

      <div
        v-if="ordenes.length === 0"
        class="orders-empty"
      >
        <h3>Aún no tienes órdenes</h3>

        <p>
          Las compras que realices aparecerán en este apartado.
        </p>
      </div>

      <div
        v-else
        class="orders-list"
      >
        <article
          v-for="orden in ordenes"
          :key="orden.id"
          class="orders-card"
        >
          <div class="orders-card__header">
            <div>
              <p>ORDEN #{{ orden.id }}</p>

              <span>
                {{ formatoFecha(orden.fecha) }}
              </span>
            </div>

            <span
              class="orders-status"
              :class="claseEstado(orden.estado)"
            >
              {{ orden.estado }}
            </span>
          </div>

          <div class="orders-products">
            <div
              v-for="(producto, index) in orden.productos"
              :key="`${orden.id}-${producto.id}-${index}`"
              class="orders-product"
            >
              <img
                :src="producto.imagen"
                :alt="producto.nombre"
              />

              <div>
                <strong>{{ producto.nombre }}</strong>

                <span>
                  Talla: {{ producto.talla || 'No aplica' }}
                </span>

                <span>
                  Cantidad: {{ producto.cantidad }}
                </span>
              </div>
              <p>
                {{
                  formatoPrecio(
                    Number(`${producto.precio}`.replace(/\D/g, '')) *
                    Number(producto.cantidad)
                )
                }}
              </p>
            </div>
          </div>

          <div class="orders-card__footer">
            <span>Total pagado</span>

            <strong>
              {{ formatoPrecio(orden.total) }}
            </strong>
          </div>
        </article>
      </div>
    </section>
  </div>
</template>

<style scoped>
.orders-overlay {
  position: fixed;
  inset: 0;
  z-index: 2800;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px;
  background: rgba(11, 11, 9, 0.72);
}

.orders-modal {
  width: 100%;
  max-width: 850px;
  max-height: 90vh;
  padding: 32px;
  overflow-y: auto;
  background: var(--ecora-cream);
}

.orders-header {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 28px;
}

.orders-header p {
  margin-bottom: 5px;
  color: var(--ecora-warm);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 3px;
}

.orders-header h2 {
  margin: 0;
  font-size: clamp(27px, 4vw, 38px);
}

.orders-close {
  border: none;
  background: transparent;
  color: var(--ecora-black);
  font-size: 32px;
  cursor: pointer;
}

.orders-empty {
  padding: 50px 20px;
  border: 1px dashed var(--ecora-sand);
  text-align: center;
}

.orders-empty h3 {
  margin-bottom: 8px;
}

.orders-empty p {
  color: var(--ecora-stone);
}

.orders-list {
  display: grid;
  gap: 20px;
}

.orders-card {
  padding: 22px;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.orders-card__header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  padding-bottom: 16px;
  border-bottom: 1px solid var(--ecora-sand);
}

.orders-card__header p {
  margin-bottom: 4px;
  color: var(--ecora-warm);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 2px;
}

.orders-card__header div span {
  color: var(--ecora-stone);
  font-size: 13px;
}

.orders-status {
  padding: 6px 10px;
  background: #f1f1f1;
  font-size: 12px;
  white-space: nowrap;
}

.orders-status--pending {
  background: #fff2cc;
  color: #765c00;
}

.orders-status--confirmed,
.orders-status--preparing {
  background: #dcecff;
  color: #174c7c;
}

.orders-status--sent {
  background: #eadcff;
  color: #53317c;
}

.orders-status--delivered {
  background: #e5f4e8;
  color: #245c2d;
}

.orders-status--cancelled {
  background: #f8d7da;
  color: #842029;
}

.orders-products {
  display: grid;
  gap: 12px;
  padding: 18px 0;
}

.orders-product {
  display: grid;
  grid-template-columns: 55px 1fr auto;
  align-items: center;
  gap: 14px;
}

.orders-product img {
  width: 55px;
  height: 65px;
  object-fit: cover;
  background: var(--ecora-sand);
}

.orders-product div {
  display: flex;
  flex-direction: column;
}

.orders-product span {
  color: var(--ecora-stone);
  font-size: 12px;
}

.orders-product p {
  margin: 0;
  font-weight: 500;
}

.orders-card__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16px;
  border-top: 1px solid var(--ecora-sand);
}

.orders-card__footer span {
  color: var(--ecora-stone);
  font-size: 13px;
}

.orders-card__footer strong {
  font-size: 21px;
}

@media (max-width: 600px) {
  .orders-overlay {
    padding: 10px;
  }

  .orders-modal {
    padding: 22px 16px;
  }

  .orders-card__header {
    flex-direction: column;
  }

  .orders-product {
    grid-template-columns: 50px 1fr;
  }

  .orders-product p {
    grid-column: 2;
  }
}
</style>