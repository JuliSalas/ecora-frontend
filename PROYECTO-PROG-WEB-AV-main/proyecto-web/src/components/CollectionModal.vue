<script setup>
import { computed, ref, watch } from 'vue'
import ProductDetalle from './ProductDetalle.vue'

const emit = defineEmits(['close', 'add-to-cart'])

const props = defineProps({
  catalogo: {
    type: Array,
    required: true,
  },
  filtroInicial: {
    type: String,
    default: 'Todos'
  }
})

const productoSeleccionado = ref(null)
const busqueda = ref('')
const filtroGenero = ref('Todos')
const filtroTipo = ref('Todos')
const ordenPrecio = ref('default')

watch(() => props.filtroInicial, (nuevo) => {
  if (nuevo) filtroTipo.value = nuevo
})

const abrirDetalle = (producto) => {
  productoSeleccionado.value = producto
}

const cerrarDetalle = () => {
  productoSeleccionado.value = null
}

const parsearPrecio = (producto) => {
  const valor = `${producto?.precio || ''}`
  const soloNumeros = valor.replace(/[^\d]/g, '')
  return Number(soloNumeros || 0)
}

const agregarAlCarrito = (item) => {
  emit('add-to-cart', item)
  cerrarDetalle()
}

const generosDisponibles = computed(() => {
  const generos = (props.catalogo || [])
    .map((producto) => producto?.categoria?.[0])
    .filter(Boolean)
    .map((valor) => valor.trim())

  return ['Todos', ...new Set(generos)]
})

const tiposDisponibles = computed(() => {
  const tipos = (props.catalogo || [])
    .map((producto) => producto?.categoria?.[1])
    .filter(Boolean)
    .map((valor) => valor.trim())

  return ['Todos', ...new Set(tipos)]
})

const productosFiltrados = computed(() => {
  let lista = [...(props.catalogo || [])]

  if (busqueda.value.trim()) {
    const texto = busqueda.value.trim().toLowerCase()
    lista = lista.filter((producto) =>
      producto?.nombre?.toLowerCase().includes(texto)
    )
  }

  if (filtroGenero.value !== 'Todos') {
    lista = lista.filter((producto) =>
      producto?.categoria?.[0]?.toLowerCase() === filtroGenero.value.toLowerCase()
    )
  }

  if (filtroTipo.value !== 'Todos') {
    lista = lista.filter((producto) =>
      producto?.categoria?.[1]?.toLowerCase() === filtroTipo.value.toLowerCase()
    )
  }

  if (ordenPrecio.value === 'asc') {
    lista.sort((a, b) => parsearPrecio(a) - parsearPrecio(b))
  } else if (ordenPrecio.value === 'desc') {
    lista.sort((a, b) => parsearPrecio(b) - parsearPrecio(a))
  }

  return lista
})
</script>

<template>
  <div class="modal-overlay" @click.self="emit('close')">
    <div class="modal-content">
      <button class="modal-close" type="button" @click="emit('close')" aria-label="Cerrar">
        ✕
      </button>

      <div style="display: flex; flex-direction: column;" class="modal-header">
        <p class="modal-eyebrow">Colección Ecora</p>
        <p class="modal-description">
          Explora piezas únicas creadas con materiales reciclados y un diseño atemporal.
        </p>
      </div>

      <div v-if="catalogo?.length" class="catalog-controls">
        <input
          v-model="busqueda"
          class="search-input"
          type="text"
          placeholder="Buscar por nombre"
        />

        <div class="filter-group">
          <select v-model="filtroGenero" class="filter-select">
            <option v-for="genero in generosDisponibles" :key="genero" :value="genero">
              {{ genero === 'Todos' ? 'Todos los géneros' : genero }}
            </option>
          </select>

          <select v-model="filtroTipo" class="filter-select">
            <option v-for="tipo in tiposDisponibles" :key="tipo" :value="tipo">
              {{ tipo === 'Todos' ? 'Todos los tipos' : tipo }}
            </option>
          </select>

          <select v-model="ordenPrecio" class="filter-select">
            <option value="default">Ordenar por</option>
            <option value="asc">Precio: menor a mayor</option>
            <option value="desc">Precio: mayor a menor</option>
          </select>
        </div>
      </div>

      <div v-if="productosFiltrados.length" class="items-grid">
        <article v-for="producto in productosFiltrados" :key="producto.nombre" class="item-card">
          <img class="product-image" :src="producto.imagen" :alt="producto.nombre">
          <div class="item-body">
            <h2>{{ producto.nombre }}</h2>
            <p class="item-price">{{ producto.precio }}</p>
            <button class="detail-button" type="button" @click="abrirDetalle(producto)">
              Ver detalles
            </button>
          </div>
        </article>
      </div>

      <div v-else class="empty-state">
        <p>No se encontraron productos con esos filtros.</p>
      </div>
    </div>
  </div>

  <ProductDetalle v-if="productoSeleccionado" :product="productoSeleccionado" @close="cerrarDetalle" @add-to-cart="agregarAlCarrito"/>
  
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@500;600&family=Jost:wght@300;400;500&display=swap');

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(11, 11, 9, 0.68);
  backdrop-filter: blur(8px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 1.5rem;
}

.modal-content {
  background: linear-gradient(145deg, #fdfbf7 0%, #f4efe7 100%);
  padding: 2.25rem;
  border-radius: 24px;
  width: min(92vw, 920px);
  max-height: 86vh;
  overflow-y: auto;
  position: relative;
  box-shadow: 0 24px 70px rgba(15, 15, 12, 0.25);
  border: 1px solid rgba(113, 108, 89, 0.16);
}

.modal-close {
  position: sticky;
  top: 1rem;
  left: calc(100% - 3rem);
  width: 2.5rem;
  height: 2.5rem;
  border: none;
  border-radius: 50%;
  display: grid;
  place-items: center;
  background: rgba(113, 108, 89, 0.12);
  color: var(--ecora-black, #0B0B09);
  font-size: 1.1rem;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease;
}

.modal-close:hover {
  transform: rotate(90deg);
  background: rgba(113, 108, 89, 0.2);
}

.modal-header {
  margin-bottom: 1.75rem;
  padding-right: 3rem;
}

.modal-eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 0.76rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  color: var(--ecora-stone, #716C59);
  margin-bottom: 0.6rem;
}

.modal-header h1 {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2rem, 3.2vw, 2.8rem);
  font-weight: 600;
  color: var(--ecora-black, #0B0B09);
  margin-bottom: 0.55rem;
}

.modal-description {
  font-family: 'Jost', sans-serif;
  color: var(--ecora-stone, #716C59);
  line-height: 1.7;
  max-width: 640px;
}

.catalog-controls {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  margin-bottom: 1.2rem;
}

.search-input,
.filter-select {
  width: 100%;
  border: 1px solid rgba(113, 108, 89, 0.2);
  border-radius: 999px;
  padding: 0.8rem 1rem;
  font-family: 'Jost', sans-serif;
  font-size: 0.95rem;
  color: var(--ecora-black, #0B0B09);
  background: rgba(255, 255, 255, 0.88);
}

.search-input:focus,
.filter-select:focus {
  outline: none;
  border-color: var(--ecora-warm, #6F5F48);
  box-shadow: 0 0 0 3px rgba(111, 95, 72, 0.12);
}

.filter-group {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.filter-select {
  flex: 1 1 180px;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
}

.item-card {
  background: rgba(255, 255, 255, 0.74);
  border: 1px solid rgba(113, 108, 89, 0.12);
  border-radius: 18px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 10px 24px rgba(11, 11, 9, 0.05);
}

.product-image {
  width: 100%;
  aspect-ratio: 4 / 5;
  object-fit: cover;
}

.item-body {
  padding: 1.1rem 1.1rem 1.25rem;
  display: flex;
  flex-direction: column;
  gap: 0.55rem;
}

.item-body h2 {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.2rem;
  color: var(--ecora-black, #0B0B09);
}

.item-price {
  font-family: 'Jost', sans-serif;
  font-size: 0.95rem;
  color: var(--ecora-stone, #716C59);
  font-weight: 500;
}

.detail-button {
  margin-top: auto;
  border: none;
  border-radius: 999px;
  padding: 0.7rem 1rem;
  background: var(--ecora-black, #0B0B09);
  color: var(--ecora-cream, #F5F3EE);
  font-family: 'Jost', sans-serif;
  font-weight: 500;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease;
}

.detail-button:hover {
  transform: translateY(-1px);
  background: var(--ecora-warm, #6F5F48);
}

.empty-state {
  padding: 2rem;
  border: 1px dashed rgba(113, 108, 89, 0.3);
  border-radius: 16px;
  text-align: center;
  color: var(--ecora-stone, #716C59);
  font-family: 'Jost', sans-serif;
}

@media (max-width: 640px) {
  .modal-content {
    padding: 1.5rem;
  }

  .modal-header {
    padding-right: 2.2rem;
  }

  .items-grid {
    grid-template-columns: 1fr;
  }
}
</style>