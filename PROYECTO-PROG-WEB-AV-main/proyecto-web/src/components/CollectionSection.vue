<script setup>
import { ref, computed, watch } from 'vue'
import ProductDetalle from './ProductDetalle.vue'

const props = defineProps({
  catalogo: {
    type: Array,
    default: () => [],
  },
  filtro: { type: String, default: 'Todos' }
})

const emit = defineEmits(['add-to-cart'])

const productoSeleccionado = ref(null)
const filtroTipo = ref('Todos')
const mostrarTodos = ref(false)

const LIMITE = 6

// Tipos únicos para los filtros
const tiposDisponibles = computed(() => {
  const tipos = (props.catalogo || [])
    .map(p => categoriaArray(p?.categoria)[1])
    .filter(Boolean)
    .map(v => v.trim())
  return ['Todos', ...new Set(tipos)]
})

const categoriaArray = (categoria) => {
  if (Array.isArray(categoria)) return categoria
  if (typeof categoria === 'string') return [categoria]
  return []
}
const productosFiltrados = computed(() => {
  if (filtroTipo.value === 'Todos') return props.catalogo
  return props.catalogo.filter(
    p => categoriaArray(p?.categoria)[1]?.toLowerCase() === filtroTipo.value.toLowerCase()
  )
})

// Solo mostrar los primeros 6 o todos
const productosVisibles = computed(() => {
  return mostrarTodos.value
    ? productosFiltrados.value
    : productosFiltrados.value.slice(0, LIMITE)
})

const hayMas = computed(() =>
  !mostrarTodos.value && productosFiltrados.value.length > LIMITE
)

watch(() => props.filtro, (nuevoFiltro) => {
  if (nuevoFiltro) {
    filtroTipo.value = nuevoFiltro
    mostrarTodos.value = false
  }
})

const abrirDetalle = (producto) => {
  productoSeleccionado.value = producto
}

const cerrarDetalle = () => {
  productoSeleccionado.value = null
}

const agregarAlCarrito = (item) => {
  emit('add-to-cart', item)
  cerrarDetalle()
}
</script>

<template>
  <section class="coleccion" id="coleccion">
    <div class="coleccion__inner">

      <!-- Encabezado -->
      <div class="coleccion__header">
        <div class="coleccion__header-left">
          <p class="coleccion__eyebrow">Nuestras piezas</p>
          <h2 class="coleccion__titulo">Colección Actual</h2>
        </div>
        <p class="coleccion__descripcion">
          Cada pieza cuenta la historia de una tela que encontró una segunda vida.
        </p>
      </div>

      <!-- Filtros por tipo -->
      <div class="coleccion__filtros">
        <button
          v-for="tipo in tiposDisponibles"
          :key="tipo"
          class="coleccion__filtro-btn"
          :class="{ 'coleccion__filtro-btn--active': filtroTipo === tipo }"
          @click="filtroTipo = tipo; mostrarTodos = false"
        >
          {{ tipo.toUpperCase() }}
        </button>
      </div>

      <!-- Grid de productos -->
      <div v-if="productosVisibles.length" class="coleccion__grid">
        <article
          v-for="producto in productosVisibles"
          :key="producto.id"
          class="coleccion__card"
          @click="abrirDetalle(producto)"
        >
          <div class="coleccion__card-img-wrap">
            <img
              :src="producto.imagen"
              :alt="producto.nombre"
              class="coleccion__card-img"
            >
            <button class="coleccion__card-agregar" type="button" @click.stop="abrirDetalle(producto)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z"/>
                <line x1="3" y1="6" x2="21" y2="6"/>
                <path d="M16 10a4 4 0 01-8 0"/>
              </svg>
              Agregar
            </button>
          </div>

          <div class="coleccion__card-body">
            <p class="coleccion__card-eyebrow">
              {{ categoriaArray(producto.categoria).join(' · ') }}
            </p>
            <h3 class="coleccion__card-nombre">{{ producto.nombre }}</h3>
            <p class="coleccion__card-precio">{{ producto.precio }}</p>
          </div>
        </article>
      </div>

      <!-- Sin resultados -->
      <div v-else class="coleccion__empty">
        <p>No hay productos en esta categoría.</p>
      </div>

      <!-- Ver más -->
      <div v-if="hayMas" class="coleccion__ver-mas">
        <button class="coleccion__ver-mas-btn" @click="mostrarTodos = true">
          Ver toda la colección
        </button>
      </div>

    </div>
  </section>

  <!-- Modal de detalle -->
  <ProductDetalle
    v-if="productoSeleccionado"
    :product="productoSeleccionado"
    @close="cerrarDetalle"
    @add-to-cart="agregarAlCarrito"
  />
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500&family=Jost:wght@300;400;500&display=swap');

.coleccion {
  background: #F5F3EE;
  padding: 6rem 2rem;
}

.coleccion__inner {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 2.5rem;
}

/* ── Encabezado ── */
.coleccion__header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 2rem;
  flex-wrap: wrap;
}

.coleccion__eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #716C59;
  margin-bottom: 0.5rem;
}

.coleccion__titulo {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2.5rem, 4vw, 3.5rem);
  font-weight: 400;
  color: #0B0B09;
  line-height: 1.1;
}

.coleccion__descripcion {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  font-weight: 300;
  color: #716C59;
  line-height: 1.7;
  max-width: 280px;
  text-align: right;
}

/* ── Filtros ── */
.coleccion__filtros {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.coleccion__filtro-btn {
  padding: 0.5rem 1.25rem;
  border: 1px solid #AEA897;
  background: transparent;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.1em;
  color: #716C59;
  cursor: pointer;
  transition: all 0.2s;
}

.coleccion__filtro-btn:hover {
  border-color: #716C59;
  color: #0B0B09;
}

.coleccion__filtro-btn--active {
  background: #716C59;
  border-color: #716C59;
  color: #F5F3EE;
}

/* ── Grid ── */
.coleccion__grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1.5rem;
}

/* ── Card ── */
.coleccion__card {
  cursor: pointer;
  display: flex;
  flex-direction: column;
}

.coleccion__card-img-wrap {
  position: relative;
  overflow: hidden;
}

.coleccion__card-img {
  width: 100%;
  aspect-ratio: 3 / 4;
  object-fit: cover;
  display: block;
  transition: transform 0.4s ease;
}

.coleccion__card:hover .coleccion__card-img {
  transform: scale(1.03);
}

/* Botón agregar sobre la imagen */
.coleccion__card-agregar {
  position: absolute;
  bottom: 1rem;
  left: 50%;
  transform: translateX(-50%) translateY(8px);
  opacity: 0;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.625rem 1.25rem;
  background: #F5F3EE;
  border: none;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #0B0B09;
  cursor: pointer;
  transition: opacity 0.25s ease, transform 0.25s ease;
  white-space: nowrap;
}

.coleccion__card:hover .coleccion__card-agregar {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

/* Body de la card */
.coleccion__card-body {
  padding: 0.875rem 0 0;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.coleccion__card-eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #AEA897;
}

.coleccion__card-nombre {
  font-family: 'Cormorant Garamond', serif;
  font-size: 18px;
  font-weight: 400;
  color: #0B0B09;
}

.coleccion__card-precio {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #716C59;
}

/* ── Empty ── */
.coleccion__empty {
  text-align: center;
  padding: 3rem;
  font-family: 'Jost', sans-serif;
  color: #AEA897;
}

/* ── Ver más ── */
.coleccion__ver-mas {
  display: flex;
  justify-content: center;
  margin-top: 1rem;
}

.coleccion__ver-mas-btn {
  padding: 0.875rem 2.5rem;
  border: 1px solid #0B0B09;
  background: transparent;
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #0B0B09;
  cursor: pointer;
  transition: all 0.2s;
}

.coleccion__ver-mas-btn:hover {
  background: #0B0B09;
  color: #F5F3EE;
}

/* ── Responsive ── */
@media (max-width: 900px) {
  .coleccion__grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .coleccion__descripcion {
    text-align: left;
    max-width: 100%;
  }
}

@media (max-width: 480px) {
  .coleccion {
    padding: 4rem 1.25rem;
  }

  .coleccion__grid {
    grid-template-columns: 1fr;
  }

  .coleccion__header {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>