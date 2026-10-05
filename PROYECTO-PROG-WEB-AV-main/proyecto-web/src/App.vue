<script setup>
import { ref, onMounted, computed, nextTick } from 'vue';
import AppHeader from './components/AppHeader.vue'
import HeroSection from './components/HeroSection.vue'
import CollectionSection from './components/CollectionSection.vue';
import NosotrosSection from './components/NosotrosSection.vue'
import ProcesoSection from './components/ProcesoSection.vue';
import ValoresSection from './components/ValoresSection.vue';
import TeamSection from './components/TeamSection.vue';
import ContactoSection from './components/ContactoSection.vue';
import AppFooter from './components/AppFooter.vue';
import CollectionModal from './components/CollectionModal.vue';
import RegisterModal from './components/RegisterModal.vue'
import LoginModal from './components/LoginModal.vue'
import CartSide from './components/CartSide.vue'
import AdminPanel from './components/AdminPanel.vue'
import UserOrdersModal from './components/UserOrdersModal.vue'

const data = ref([])
const error = ref('')
const isAdminPanelOpen = ref(false)
const isUserOrdersOpen = ref(false)
const productos = ref([])
const ordenes = ref([])

onMounted(async () => {

  try {
  const response = await fetch('/api/productos')

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`)
  }

  const productosAPI = await response.json()

  productos.value = productosAPI.map((producto) => ({
    ...producto,
    id: producto._id,

    categoria: producto.categorias.map(
      (categoria) => `${categoria.genero} - ${categoria.tipo}`
    ),

    inventario: producto.inventario ?? 10
  }))

  } catch (err) {
    error.value = 'No Products found! :('
    console.error(err)
  }

  const sesionGuardada = localStorage.getItem('ecoraSesion')

  if (sesionGuardada) {
    try {
      usuarioSesion.value = JSON.parse(sesionGuardada)
    } catch (errorSesion) {
      console.error(
        'No fue posible recuperar la sesión:',
        errorSesion
      )

      localStorage.removeItem('ecoraSesion')
      usuarioSesion.value = null
    }
  }
})

const isModalOpen = ref(false)
const isRegisterOpen = ref(false)

const manejarRegistroExitoso = (nuevoUsuario) => {
  console.log('Usuario registrado:', nuevoUsuario)
}

const isLoginOpen = ref(false)
const usuarioSesion = ref(null)

const manejarInicioSesion = (usuario) => {
  usuarioSesion.value = usuario
  isLoginOpen.value = false
}

const cerrarSesion = () => {
  localStorage.removeItem('ecoraSesion')
  localStorage.removeItem('ecoraToken')
  
  usuarioSesion.value = null
  isAdminPanelOpen.value = false
  isUserOrdersOpen.value = false
}

// ── Estado del carrito ──
const cartOpen = ref(false)
const cartItems = ref([])

const addToCart = (item) => {
  // Buscar si ya existe el mismo producto con la misma talla
  const existing = cartItems.value.find(
    i => i.id === item.id && i.talla === item.talla
  )

  if (existing) {
    existing.cantidad += item.cantidad
  } else {
    cartItems.value.push({ ...item })
  }
}

const removeFromCart = (index) => {
  cartItems.value.splice(index, 1)
}

const cartCount = computed(() =>
  cartItems.value.reduce(
    (sum, i) => sum + Number(i.cantidad || 0),
    0
  )
)

const obtenerPrecioNumerico = (precio) => {
  if (typeof precio === 'number') {
    return precio
  }

  if (typeof precio === 'string') {
    return Number(precio.replace(/\D/g, '')) || 0
  }

  return 0
}

const cartTotal = computed(() =>
  cartItems.value.reduce((sum, i) => {
    const precio = obtenerPrecioNumerico(i.precio)

    return sum + precio * Number(i.cantidad || 0)
  }, 0)
)

const clearCart = () => {
  cartItems.value = []
}

const filtroColeccion = ref('Todos')

const aplicarFiltro = async (tipo) => {
  filtroColeccion.value = tipo
  await nextTick()

  document
    .querySelector('#coleccion')
    ?.scrollIntoView({ behavior: 'smooth' })
}

const abrirPanelAdministrador = async () => {
  if (usuarioSesion.value?.rol !== 'administrador') {
    return
  }

  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const response = await fetch('/api/ordenes', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`
      }
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj ||
        resultado.msg ||
        'No fue posible obtener las órdenes.'
      )
    }

    ordenes.value = resultado.map((orden) => ({
      ...orden,

      // Usamos el _id de Mongo para poder actualizar la orden
      id: orden._id,

      // Conservamos el número original para mostrarlo
      numeroOrden: orden.id,

      cliente: {
        id: orden.clienteAsociado?.[0]?.cliente?._id,
        nombre:
          orden.clienteAsociado?.[0]?.cliente?.nombre ||
          'Cliente',
        correo:
          orden.clienteAsociado?.[0]?.cliente?.correo ||
          ''
      },

      productos: orden.productosOrden.map((item) => ({
        ...item.productos,
        id: item.productos?._id,
        talla: item.talla || 'No aplica',
        cantidad: item.cantidad || 1
      }))
    }))

    console.log('ÓRDENES ADMIN:', ordenes.value)

    isAdminPanelOpen.value = true

  } catch (error) {
    console.error('Error al cargar órdenes del administrador:', error)
  }
}

const registrarProducto = async (nuevoProducto) => {
  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const categorias = nuevoProducto.categoria.map((categoria) => {
      const partes = categoria.split('-').map((parte) => parte.trim())

      return {
        genero: partes[0],
        tipo: partes[1]
      }
    })

    const response = await fetch('/api/productos', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        nombre: nuevoProducto.nombre,
        categorias: categorias,
        tallas: nuevoProducto.tallas,
        descripcion: nuevoProducto.descripcion,
        precio: `CRC ${Number(nuevoProducto.precio).toLocaleString('es-CR')}`,
        imagen: nuevoProducto.imagen
      })
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj || 'No fue posible registrar el producto.'
      )
    }

    console.log('PRODUCTO REGISTRADO:', resultado)

    const productoGuardado = resultado.producto

    productos.value.push({
      ...productoGuardado,
      id: productoGuardado._id,
      categoria: categorias.map(
        (categoria) => `${categoria.genero} - ${categoria.tipo}`),
      inventario: nuevoProducto.inventario
    })

  } catch (error) {
    console.error('Error al registrar producto:', error)
  }
}

const editarProducto = async (productoEditado) => {
  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const categorias = productoEditado.categoria.map((categoria) => {
      const partes = categoria.split('-').map((parte) => parte.trim())

      return {
        genero: partes[0],
        tipo: partes[1]
      }
    })

    const response = await fetch(`/api/productos/${productoEditado.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        nombre: productoEditado.nombre,
        categorias: categorias,
        tallas: productoEditado.tallas,
        descripcion: productoEditado.descripcion,
        precio: `CRC ${Number(productoEditado.precio).toLocaleString('es-CR')}`,
        imagen: productoEditado.imagen
      })
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj || 'No fue posible actualizar el producto.'
      )
    }

    console.log('PRODUCTO ACTUALIZADO:', resultado)

    const productoActualizado = resultado.producto

    const indice = productos.value.findIndex(
      (producto) => producto.id === productoEditado.id
    )

    if (indice !== -1) {
      productos.value[indice] = {
        ...productoActualizado,
        id: productoActualizado._id,
        categoria: productoActualizado.categorias.map(
          (categoria) => `${categoria.genero} - ${categoria.tipo}`),
        inventario: productoEditado.inventario
      }
    }

  } catch (error) {
    console.error('Error al editar producto:', error)
  }
}

const eliminarProducto = async (productoId) => {
  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const response = await fetch(`/api/productos/${productoId}`, {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${token}`
      }
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj || 'No fue posible eliminar el producto.'
      )
    }

    console.log('PRODUCTO ELIMINADO:', resultado)

    productos.value = productos.value.filter(
      (producto) => producto.id !== productoId
    )

  } catch (error) {
    console.error('Error al eliminar producto:', error)
  }
}

// ── Estado de órdenes ──

const registrarOrden = () => {
  clearCart()
  cartOpen.value = false
}

const actualizarEstadoOrden = async ({ id, estado }) => {
  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      throw new Error('No hay una sesión válida.')
    }

    const response = await fetch(`/api/ordenes/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({
        estado
      })
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj ||
        'No fue posible actualizar el estado de la orden.'
      )
    }

    console.log('ESTADO DE ORDEN ACTUALIZADO:', resultado)

    const indice = ordenes.value.findIndex(
      (orden) => orden.id === id
    )

    if (indice !== -1) {
      ordenes.value[indice] = {
        ...ordenes.value[indice],
        estado: resultado.orden.estado
      }
    }

  } catch (error) {
    console.error('Error al actualizar estado de orden:', error)
  }
}

const cargarMisOrdenes = async () => {
  try {
    const token = localStorage.getItem('ecoraToken')

    if (!token) {
      console.error('No hay token de sesión.')
      return
    }

    const response = await fetch('/api/ordenes/mis-compras', {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${token}`
      }
    })

    const resultado = await response.json()

    if (!response.ok) {
      throw new Error(
        resultado.msj ||
        resultado.msg ||
        'No fue posible consultar las compras.'
      )
    }

    ordenes.value = resultado.map((orden) => ({
      ...orden,

      productos: orden.productosOrden.map((item) => ({
        ...item.productos,
        id: item.productos._id,

        // La orden actual no guarda talla/cantidad en MongoDB,
        // asi que dejo valores compatibles con el modal.
        talla: item.talla || 'No aplica',
        cantidad: item.cantidad || 1
      }))
    }))

    isUserOrdersOpen.value = true

  } catch (error) {
    console.error('Error al cargar las órdenes:', error)
  }
}
</script>

<template>
  <AppHeader
    :usuario="usuarioSesion"
    :cart-count="cartCount"
    @open-register="isRegisterOpen = true"
    @open-login="isLoginOpen = true"
    @open-admin="abrirPanelAdministrador"
    @open-orders="cargarMisOrdenes"
    @logout="cerrarSesion"
    @open-cart="cartOpen = true"
  />

  <HeroSection
    @open-collection="isModalOpen = true"
  />

  <CollectionSection
    :catalogo="productos"
    :filtro="filtroColeccion"
    @add-to-cart="addToCart"
  />

  <NosotrosSection />
  <ValoresSection />
  <TeamSection />
  <ProcesoSection />
  <ContactoSection />

  <AppFooter
    @filtrar-coleccion="aplicarFiltro"
  />

  <CollectionModal
    v-if="isModalOpen"
    :catalogo="productos"
    :filtro-inicial="filtroColeccion"
    @close="isModalOpen = false"
    @add-to-cart="addToCart"
  />

  <CartSide
    v-if="cartOpen"
    :items="cartItems"
    :total="cartTotal"
    :usuario="usuarioSesion"
    @close="cartOpen = false"
    @remove-item="removeFromCart"
    @cart-paid="registrarOrden"
  />

  <RegisterModal
    v-if="isRegisterOpen"
    @close="isRegisterOpen = false"
    @registered="manejarRegistroExitoso"
  />

  <LoginModal
    v-if="isLoginOpen"
    @close="isLoginOpen = false"
    @login-success="manejarInicioSesion"
  />

  <UserOrdersModal
    v-if="isUserOrdersOpen && usuarioSesion"
    :ordenes="ordenes"
    @close="isUserOrdersOpen = false"
  />

  <AdminPanel
    v-if="
      isAdminPanelOpen &&
      usuarioSesion?.rol === 'administrador'
    "
    :productos="productos"
    :ordenes="ordenes"
    @close="isAdminPanelOpen = false"
    @registrar-producto="registrarProducto"
    @editar-producto="editarProducto"
    @eliminar-producto="eliminarProducto"
    @actualizar-estado-orden="actualizarEstadoOrden"
  />
</template>

<style>
/* Reset global y variables de Ecora */
*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --ecora-cream: #F5F3EE;
  --ecora-stone: #716C59;
  --ecora-sand: #AEA897;
  --ecora-black: #0B0B09;
  --ecora-warm: #6F5F48;
  --ecora-gray: #A5A8AF;
  --ecora-error: #c0392b;
}

body {
  background: var(--ecora-cream);
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  -webkit-font-smoothing: antialiased;
}

img {
  max-width: 100%;
  display: block;
}

a {
  text-decoration: none;
  color: inherit;
}

.form-input,
.form-group {
  min-width: 0;
  box-sizing: border-box;
}
</style>