<script setup>
import { computed, reactive, ref } from 'vue'

const props = defineProps({
  productos: {
    type: Array,
    default: () => []
  },
  ordenes: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits([
  'close',
  'registrar-producto',
  'editar-producto',
  'eliminar-producto',
  'actualizar-estado-orden'
])

const seccionActiva = ref('resumen')
const mostrarFormulario = ref(false)
const modoEdicion = ref(false)
const productoEditandoId = ref(null)
const mensaje = ref('')

const formulario = reactive({
  nombre: '',
  categoria: '',
  descripcion: '',
  precio: '',
  inventario: '',
  imagen: '',
  tallas: ''
})

const errores = reactive({
  nombre: '',
  categoria: '',
  descripcion: '',
  precio: '',
  inventario: '',
  imagen: ''
})

const estadosOrden = [
  'Pendiente',
  'Confirmada',
  'En preparación',
  'Enviada',
  'Entregada',
  'Cancelada'
]

const totalProductos = computed(() => {
  return props.productos.length
})

const totalOrdenes = computed(() => {
  return props.ordenes.length
})

const ordenesPendientes = computed(() => {
  return props.ordenes.filter(
    (orden) => orden.estado === 'Pendiente'
  ).length
})

const productosInventarioBajo = computed(() => {
  return props.productos.filter((producto) => {
    const inventario = Number(producto.inventario)

    return inventario >= 0 && inventario <= 5
  }).length
})

const cambiarSeccion = (seccion) => {
  seccionActiva.value = seccion
  mensaje.value = ''
  cerrarFormulario()
}

const limpiarErrores = () => {
  errores.nombre = ''
  errores.categoria = ''
  errores.descripcion = ''
  errores.precio = ''
  errores.inventario = ''
  errores.imagen = ''
}

const limpiarFormulario = () => {
  formulario.nombre = ''
  formulario.categoria = ''
  formulario.descripcion = ''
  formulario.precio = ''
  formulario.inventario = ''
  formulario.imagen = ''
  formulario.tallas = ''

  productoEditandoId.value = null
  modoEdicion.value = false

  limpiarErrores()
}

const abrirFormularioNuevo = () => {
  limpiarFormulario()
  mensaje.value = ''
  mostrarFormulario.value = true
}

const cerrarFormulario = () => {
  mostrarFormulario.value = false
  limpiarFormulario()
}

const validarFormulario = () => {
  limpiarErrores()

  let formularioValido = true

  if (!formulario.nombre.trim()) {
    errores.nombre = 'El nombre del producto es obligatorio.'
    formularioValido = false
  } else if (formulario.nombre.trim().length < 3) {
    errores.nombre =
      'El nombre debe contener al menos 3 caracteres.'
    formularioValido = false
  }

  if (!formulario.categoria.trim()) {
    errores.categoria = 'La categoría es obligatoria.'
    formularioValido = false
  }

  if (!formulario.descripcion.trim()) {
    errores.descripcion = 'La descripción es obligatoria.'
    formularioValido = false
  } else if (formulario.descripcion.trim().length < 10) {
    errores.descripcion =
      'La descripción debe contener al menos 10 caracteres.'
    formularioValido = false
  }

  if (
    formulario.precio === '' ||
    formulario.precio === null ||
    Number(formulario.precio) <= 0
  ) {
    errores.precio = 'Ingrese un precio mayor que cero.'
    formularioValido = false
  }

  if (
    formulario.inventario === '' ||
    formulario.inventario === null ||
    Number(formulario.inventario) < 0
  ) {
    errores.inventario =
      'El inventario debe ser igual o mayor que cero.'
    formularioValido = false
  }

  if (!formulario.imagen.trim()) {
    errores.imagen = 'La ruta de la imagen es obligatoria.'
    formularioValido = false
  }

  return formularioValido
}

const construirProducto = () => {
  const tallas = formulario.tallas
    .split(',')
    .map((talla) => talla.trim())
    .filter(Boolean)

  const categorias = formulario.categoria
    .split(',')
    .map((categoria) => categoria.trim())
    .filter(Boolean)

  return {
    nombre: formulario.nombre.trim(),
    categoria: categorias,
    descripcion: formulario.descripcion.trim(),
    precio: Number(formulario.precio),
    inventario: Number(formulario.inventario),
    imagen: formulario.imagen.trim(),
    tallas
  }
}

const guardarProducto = () => {
  if (!validarFormulario()) {
    return
  }

  const datosProducto = construirProducto()

  if (modoEdicion.value) {
    emit('editar-producto', {
      ...datosProducto,
      id: productoEditandoId.value
    })

    mensaje.value =
      'El producto fue actualizado correctamente.'
  } else {
    emit('registrar-producto', datosProducto)

    mensaje.value =
      'El producto fue registrado correctamente.'
  }

  cerrarFormulario()
}

const prepararEdicion = (producto) => {
  productoEditandoId.value = producto.id
  modoEdicion.value = true

  formulario.nombre = producto.nombre ?? ''
  formulario.categoria = Array.isArray(producto.categoria)
  ? producto.categoria.join(', ')
  : producto.categoria ?? ''
  formulario.descripcion = producto.descripcion ?? ''
  formulario.precio = producto.precio ?? ''
  formulario.inventario = producto.inventario ?? 0
  formulario.imagen = producto.imagen ?? ''

  formulario.tallas = Array.isArray(producto.tallas)
    ? producto.tallas.join(', ')
    : ''

  mensaje.value = ''
  mostrarFormulario.value = true
}

const confirmarEliminacion = (producto) => {
  const confirmado = window.confirm(
    `¿Desea eliminar el producto "${producto.nombre}"?`
  )

  if (!confirmado) {
    return
  }

  emit('eliminar-producto', producto.id)

  mensaje.value =
    'El producto fue eliminado correctamente.'
}

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

const obtenerEstadoInventario = (inventario) => {
  const cantidad = Number(inventario)

  if (cantidad === 0) {
    return 'Agotado'
  }

  if (cantidad <= 5) {
    return 'Inventario bajo'
  }

  return 'Disponible'
}

const manejarErrorImagen = (evento) => {
  evento.target.onerror = null
  evento.target.src = '/placeholder-producto.jpg'
}

const cambiarEstadoOrden = (orden, estado) => {
  emit('actualizar-estado-orden', {
    id: orden.id,
    estado
  })
}

const cerrarPanel = () => {
  emit('close')
}
</script>

<template>
  <div class="admin-panel">
    <aside class="admin-sidebar">
      <div class="admin-sidebar__header">
        <p>ECORA</p>
        <h2>Administración</h2>
      </div>

      <nav class="admin-sidebar__nav">
        <button
          type="button"
          :class="{ activo: seccionActiva === 'resumen' }"
          @click="cambiarSeccion('resumen')"
        >
          Resumen
        </button>

        <button
          type="button"
          :class="{ activo: seccionActiva === 'productos' }"
          @click="cambiarSeccion('productos')"
        >
          Productos
        </button>

        <button
          type="button"
          :class="{ activo: seccionActiva === 'ordenes' }"
          @click="cambiarSeccion('ordenes')"
        >
          Órdenes
        </button>
      </nav>

      <button
        class="admin-sidebar__close"
        type="button"
        @click="cerrarPanel"
      >
        Volver a la tienda
      </button>
    </aside>

    <main class="admin-content">
      <!-- RESUMEN -->
      <section v-if="seccionActiva === 'resumen'">
        <div class="admin-heading">
          <p>PANEL DE CONTROL</p>
          <h1>Resumen administrativo</h1>
        </div>

        <div class="admin-cards">
          <article class="admin-card">
            <span>Productos</span>
            <strong>{{ totalProductos }}</strong>
          </article>

          <article class="admin-card">
            <span>Órdenes</span>
            <strong>{{ totalOrdenes }}</strong>
          </article>

          <article class="admin-card">
            <span>Órdenes pendientes</span>
            <strong>{{ ordenesPendientes }}</strong>
          </article>

          <article class="admin-card">
            <span>Inventario bajo</span>
            <strong>{{ productosInventarioBajo }}</strong>
          </article>
        </div>

        <div class="admin-summary">
          <h2>Estado general de la tienda</h2>

          <p v-if="totalProductos === 0">
            Todavía no existen productos registrados.
          </p>

          <p v-else>
            El catálogo cuenta con
            <strong>{{ totalProductos }}</strong>
            productos.
          </p>

          <p>
            Se han generado
            <strong>{{ totalOrdenes }}</strong>
            órdenes, de las cuales
            <strong>{{ ordenesPendientes }}</strong>
            están pendientes.
          </p>

          <p v-if="productosInventarioBajo > 0">
            Existen
            <strong>{{ productosInventarioBajo }}</strong>
            productos agotados o con inventario bajo.
          </p>
        </div>
      </section>

      <!-- PRODUCTOS -->
      <section v-else-if="seccionActiva === 'productos'">
        <div class="admin-heading admin-heading--actions">
          <div>
            <p>CATÁLOGO</p>
            <h1>Gestión de productos</h1>
          </div>

          <button
            class="admin-primary-button"
            type="button"
            @click="abrirFormularioNuevo"
          >
            Registrar producto
          </button>
        </div>

        <div
          v-if="mensaje"
          class="alert alert-success"
          role="alert"
        >
          {{ mensaje }}
        </div>

        <form
          v-if="mostrarFormulario"
          class="admin-product-form"
          novalidate
          @submit.prevent="guardarProducto"
        >
          <div class="admin-product-form__header">
            <div>
              <p>FORMULARIO</p>

              <h2>
                {{
                  modoEdicion
                    ? 'Editar producto'
                    : 'Registrar producto'
                }}
              </h2>
            </div>

            <button
              class="admin-close-form"
              type="button"
              @click="cerrarFormulario"
            >
              Cancelar
            </button>
          </div>

          <div class="admin-form-grid">
            <div>
              <label for="adminNombre" class="form-label">
                Nombre
              </label>

              <input
                id="adminNombre"
                v-model="formulario.nombre"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errores.nombre }"
                placeholder="Nombre del producto"
              />

              <div class="invalid-feedback">
                {{ errores.nombre }}
              </div>
            </div>

            <div>
              <label for="adminCategoria" class="form-label">
                Categoría
              </label>

              <input
                id="adminCategoria"
                v-model="formulario.categoria"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errores.categoria }"
                placeholder="Ejemplo: Camisas"
              />

              <div class="invalid-feedback">
                {{ errores.categoria }}
              </div>
            </div>

            <div>
              <label for="adminPrecio" class="form-label">
                Precio
              </label>

              <input
                id="adminPrecio"
                v-model.number="formulario.precio"
                type="number"
                min="1"
                step="1"
                class="form-control"
                :class="{ 'is-invalid': errores.precio }"
                placeholder="Precio en colones"
              />

              <div class="invalid-feedback">
                {{ errores.precio }}
              </div>
            </div>

            <div>
              <label for="adminInventario" class="form-label">
                Inventario
              </label>

              <input
                id="adminInventario"
                v-model.number="formulario.inventario"
                type="number"
                min="0"
                step="1"
                class="form-control"
                :class="{ 'is-invalid': errores.inventario }"
                placeholder="Cantidad disponible"
              />

              <div class="invalid-feedback">
                {{ errores.inventario }}
              </div>
            </div>

            <div class="admin-form-grid__full">
              <label for="adminImagen" class="form-label">
                Ruta de imagen
              </label>

              <input
                id="adminImagen"
                v-model="formulario.imagen"
                type="text"
                class="form-control"
                :class="{ 'is-invalid': errores.imagen }"
                placeholder="/productos/producto.jpg"
              />

              <div class="invalid-feedback">
                {{ errores.imagen }}
              </div>

              <small class="admin-help">
                La imagen debe encontrarse dentro de la carpeta
                public del proyecto.
              </small>
            </div>

            <div class="admin-form-grid__full">
              <label for="adminTallas" class="form-label">
                Tallas
              </label>

              <input
                id="adminTallas"
                v-model="formulario.tallas"
                type="text"
                class="form-control"
                placeholder="S, M, L, XL"
              />

              <small class="admin-help">
                Separe las tallas utilizando comas.
              </small>
            </div>

            <div class="admin-form-grid__full">
              <label for="adminDescripcion" class="form-label">
                Descripción
              </label>

              <textarea
                id="adminDescripcion"
                v-model="formulario.descripcion"
                class="form-control"
                :class="{ 'is-invalid': errores.descripcion }"
                rows="4"
                placeholder="Descripción del producto"
              ></textarea>

              <div class="invalid-feedback">
                {{ errores.descripcion }}
              </div>
            </div>
          </div>

          <button
            class="admin-primary-button"
            type="submit"
          >
            {{
              modoEdicion
                ? 'Guardar cambios'
                : 'Registrar producto'
            }}
          </button>
        </form>

        <div
          v-if="productos.length === 0"
          class="admin-empty"
        >
          No hay productos registrados.
        </div>

        <div
          v-else
          class="admin-table-container"
        >
          <table class="admin-table">
            <thead>
              <tr>
                <th>Producto</th>
                <th>Categoría</th>
                <th>Precio</th>
                <th>Inventario</th>
                <th>Estado</th>
                <th>Acciones</th>
              </tr>
            </thead>

            <tbody>
              <tr
                v-for="producto in productos"
                :key="producto.id"
              >
                <td>
                  <div class="admin-product-cell">
                    <img
                      :src="producto.imagen"
                      :alt="producto.nombre"
                      @error="manejarErrorImagen"
                    />

                    <div>
                      <strong>{{ producto.nombre }}</strong>
                      <small>ID: {{ producto.id }}</small>
                    </div>
                  </div>
                </td>

                <td>{{ producto.categoria }}</td>
                <td>{{ formatoPrecio(producto.precio) }}</td>
                <td>{{ producto.inventario }}</td>

                <td>
                  <span
                    class="admin-stock"
                    :class="{
                      'admin-stock--empty':
                        Number(producto.inventario) === 0,
                      'admin-stock--low':
                        Number(producto.inventario) > 0 &&
                        Number(producto.inventario) <= 5
                    }"
                  >
                    {{
                      obtenerEstadoInventario(
                        producto.inventario
                      )
                    }}
                  </span>
                </td>

                <td>
                  <div class="admin-table-actions">
                    <button
                      class="admin-edit-button"
                      type="button"
                      @click="prepararEdicion(producto)"
                    >
                      Editar
                    </button>

                    <button
                      class="admin-delete-button"
                      type="button"
                      @click="confirmarEliminacion(producto)"
                    >
                      Eliminar
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- ÓRDENES -->
      <section v-else-if="seccionActiva === 'ordenes'">
        <div class="admin-heading">
          <p>VENTAS</p>
          <h1>Seguimiento de órdenes</h1>
        </div>

        <div
          v-if="ordenes.length === 0"
          class="admin-empty"
        >
          Todavía no se han generado órdenes.
        </div>

        <div
          v-else
          class="admin-orders"
        >
          <article
            v-for="orden in ordenes"
            :key="orden.id"
            class="admin-order"
          >
            <div class="admin-order__header">
              <div>
                <p class="admin-order__label">
                  ORDEN #{{ orden.numeroOrden || orden.id }}
                </p>

                <h2>
                  {{ orden.cliente?.nombre || 'Cliente' }}
                </h2>

                <span>
                  {{ orden.cliente?.correo }}
                </span>
              </div>

              <div class="admin-order__date">
                {{ formatoFecha(orden.fecha) }}
              </div>
            </div>

            <div class="admin-order__products">
              <div
                v-for="(producto, index) in orden.productos"
                :key="`${orden.id}-${producto.id}-${index}`"
                class="admin-order-product"
              >
                <img
                  :src="producto.imagen"
                  :alt="producto.nombre"
                  @error="manejarErrorImagen"
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
                      (Number(`${producto.precio}`.replace(/\D/g, '')) || 0) *
                      Number(producto.cantidad || 1)
                    )
                  }}
                </p>
              </div>
            </div>

            <div class="admin-order__footer">
              <div>
                <span>Total</span>
                <strong>{{ formatoPrecio(orden.total) }}</strong>
              </div>

              <div class="admin-order__status">
                <label :for="`estado-${orden.id}`">
                  Estado
                </label>

                <select
                  :id="`estado-${orden.id}`"
                  class="form-select"
                  :value="orden.estado"
                  @change="
                    cambiarEstadoOrden(
                      orden,
                      $event.target.value
                    )
                  "
                >
                  <option
                    v-for="estado in estadosOrden"
                    :key="estado"
                    :value="estado"
                  >
                    {{ estado }}
                  </option>
                </select>
              </div>
            </div>
          </article>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.admin-panel {
  position: fixed;
  inset: 0;
  z-index: 3000;
  display: grid;
  grid-template-columns: 260px minmax(0, 1fr);
  min-height: 100vh;
  background: var(--ecora-cream);
}

.admin-sidebar {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  padding: 32px 22px;
  background: var(--ecora-black);
  color: var(--ecora-cream);
}

.admin-sidebar__header {
  margin-bottom: 42px;
}

.admin-sidebar__header p {
  margin-bottom: 6px;
  color: var(--ecora-sand);
  font-size: 11px;
  letter-spacing: 4px;
}

.admin-sidebar__header h2 {
  margin: 0;
  color: var(--ecora-cream);
  font-size: 24px;
}

.admin-sidebar__nav {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.admin-sidebar__nav button {
  padding: 13px 14px;
  border: none;
  border-left: 3px solid transparent;
  background: transparent;
  color: var(--ecora-cream);
  font-family: inherit;
  text-align: left;
  cursor: pointer;
  transition:
    background 0.2s ease,
    border-color 0.2s ease;
}

.admin-sidebar__nav button:hover,
.admin-sidebar__nav button.activo {
  border-left-color: var(--ecora-sand);
  background: rgba(255, 255, 255, 0.08);
}

.admin-sidebar__close {
  margin-top: auto;
  padding: 12px;
  border: 1px solid var(--ecora-sand);
  background: transparent;
  color: var(--ecora-cream);
  font-family: inherit;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.admin-sidebar__close:hover {
  background: var(--ecora-cream);
  color: var(--ecora-black);
}

.admin-content {
  padding: 48px;
  overflow-y: auto;
}

.admin-heading {
  margin-bottom: 30px;
}

.admin-heading p {
  margin-bottom: 8px;
  color: var(--ecora-warm);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 3px;
}

.admin-heading h1 {
  margin: 0;
  font-size: clamp(30px, 4vw, 45px);
}

.admin-heading--actions {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 20px;
}

.admin-cards {
  display: grid;
  grid-template-columns: repeat(4, minmax(170px, 1fr));
  gap: 20px;
}

.admin-card {
  display: flex;
  flex-direction: column;
  min-height: 145px;
  padding: 24px;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.admin-card span {
  color: var(--ecora-stone);
  font-size: 13px;
}

.admin-card strong {
  margin-top: auto;
  font-size: 38px;
  font-weight: 500;
}

.admin-summary {
  margin-top: 30px;
  padding: 26px;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.admin-summary h2 {
  margin-bottom: 15px;
  font-size: 24px;
}

.admin-summary p {
  margin-bottom: 8px;
  color: var(--ecora-stone);
}

.admin-primary-button {
  padding: 12px 20px;
  border: 1px solid var(--ecora-black);
  background: var(--ecora-black);
  color: var(--ecora-cream);
  font-family: inherit;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 1px;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.admin-primary-button:hover {
  background: transparent;
  color: var(--ecora-black);
}

.admin-product-form {
  margin-bottom: 32px;
  padding: 26px;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.admin-product-form__header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 24px;
}

.admin-product-form__header p {
  margin-bottom: 4px;
  color: var(--ecora-warm);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 2px;
}

.admin-product-form__header h2 {
  margin: 0;
  font-size: 25px;
}

.admin-close-form {
  border: none;
  background: transparent;
  color: var(--ecora-stone);
  cursor: pointer;
}

.admin-close-form:hover {
  color: var(--ecora-black);
  text-decoration: underline;
}

.admin-form-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 20px;
  margin-bottom: 24px;
}

.admin-form-grid__full {
  grid-column: 1 / -1;
}

.form-label {
  font-weight: 500;
}

.form-control,
.form-select {
  min-height: 46px;
  border: 1px solid var(--ecora-sand);
  border-radius: 2px;
}

.form-control:focus,
.form-select:focus {
  border-color: var(--ecora-warm);
  box-shadow: 0 0 0 0.2rem rgba(111, 95, 72, 0.15);
}

textarea.form-control {
  min-height: 110px;
  resize: vertical;
}

.admin-help {
  display: block;
  margin-top: 6px;
  color: var(--ecora-stone);
}

.admin-table-container {
  overflow-x: auto;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.admin-table {
  width: 100%;
  min-width: 850px;
  border-collapse: collapse;
}

.admin-table th,
.admin-table td {
  padding: 15px;
  border-bottom: 1px solid var(--ecora-sand);
  text-align: left;
  vertical-align: middle;
}

.admin-table th {
  background: rgba(0, 0, 0, 0.035);
  font-size: 11px;
  letter-spacing: 1px;
  text-transform: uppercase;
}

.admin-table tbody tr:last-child td {
  border-bottom: none;
}

.admin-product-cell {
  display: flex;
  align-items: center;
  gap: 12px;
}

.admin-product-cell img {
  width: 52px;
  height: 62px;
  flex-shrink: 0;
  object-fit: cover;
  background: var(--ecora-sand);
}

.admin-product-cell div {
  display: flex;
  flex-direction: column;
}

.admin-product-cell small {
  margin-top: 4px;
  color: var(--ecora-stone);
}

.admin-stock {
  display: inline-block;
  padding: 5px 9px;
  background: #e5f4e8;
  color: #245c2d;
  font-size: 11px;
  white-space: nowrap;
}

.admin-stock--low {
  background: #fff2cc;
  color: #765c00;
}

.admin-stock--empty {
  background: #f8d7da;
  color: #842029;
}

.admin-table-actions {
  display: flex;
  gap: 8px;
}

.admin-edit-button,
.admin-delete-button {
  padding: 7px 11px;
  background: transparent;
  font-family: inherit;
  font-size: 11px;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.admin-edit-button {
  border: 1px solid var(--ecora-warm);
  color: var(--ecora-warm);
}

.admin-delete-button {
  border: 1px solid #9c3434;
  color: #9c3434;
}

.admin-edit-button:hover {
  background: var(--ecora-warm);
  color: #fff;
}

.admin-delete-button:hover {
  background: #9c3434;
  color: #fff;
}

.admin-empty {
  padding: 40px;
  border: 1px dashed var(--ecora-sand);
  text-align: center;
  color: var(--ecora-stone);
}

.admin-orders {
  display: grid;
  gap: 22px;
}

.admin-order {
  padding: 24px;
  border: 1px solid var(--ecora-sand);
  background: #fff;
}

.admin-order__header {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  padding-bottom: 18px;
  border-bottom: 1px solid var(--ecora-sand);
}

.admin-order__label {
  margin-bottom: 5px;
  color: var(--ecora-warm);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 2px;
}

.admin-order__header h2 {
  margin-bottom: 4px;
  font-size: 24px;
}

.admin-order__header span,
.admin-order__date {
  color: var(--ecora-stone);
  font-size: 13px;
}

.admin-order__products {
  display: grid;
  gap: 12px;
  padding: 18px 0;
}

.admin-order-product {
  display: grid;
  grid-template-columns: 55px 1fr auto;
  align-items: center;
  gap: 14px;
}

.admin-order-product img {
  width: 55px;
  height: 65px;
  object-fit: cover;
  background: var(--ecora-sand);
}

.admin-order-product div {
  display: flex;
  flex-direction: column;
}

.admin-order-product span {
  color: var(--ecora-stone);
  font-size: 12px;
}

.admin-order-product p {
  margin: 0;
  font-weight: 500;
}

.admin-order__footer {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 24px;
  padding-top: 18px;
  border-top: 1px solid var(--ecora-sand);
}

.admin-order__footer > div:first-child {
  display: flex;
  flex-direction: column;
}

.admin-order__footer span,
.admin-order__status label {
  color: var(--ecora-stone);
  font-size: 12px;
}

.admin-order__footer strong {
  font-size: 24px;
}

.admin-order__status {
  width: min(100%, 240px);
}

.admin-order__status label {
  display: block;
  margin-bottom: 5px;
}

@media (max-width: 1100px) {
  .admin-cards {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 900px) {
  .admin-panel {
    display: block;
    overflow-y: auto;
  }

  .admin-sidebar {
    min-height: auto;
  }

  .admin-sidebar__nav {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }

  .admin-sidebar__close {
    margin-top: 22px;
  }

  .admin-content {
    padding: 30px 20px;
    overflow: visible;
  }

  .admin-heading--actions {
    align-items: flex-start;
  }

  .admin-form-grid {
    grid-template-columns: 1fr;
  }

  .admin-form-grid__full {
    grid-column: auto;
  }
}

@media (max-width: 600px) {
  .admin-sidebar__nav {
    grid-template-columns: 1fr;
  }

  .admin-cards {
    grid-template-columns: 1fr;
  }

  .admin-heading--actions,
  .admin-order__header,
  .admin-order__footer {
    flex-direction: column;
    align-items: stretch;
  }

  .admin-primary-button {
    width: 100%;
  }

  .admin-product-form {
    padding: 20px 15px;
  }

  .admin-product-form__header {
    flex-direction: column;
  }

  .admin-order-product {
    grid-template-columns: 50px 1fr;
  }

  .admin-order-product p {
    grid-column: 2;
  }

  .admin-order__status {
    width: 100%;
  }
}
</style>
