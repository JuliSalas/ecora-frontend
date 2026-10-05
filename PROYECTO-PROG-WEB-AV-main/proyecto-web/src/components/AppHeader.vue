<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits([
  'toggle-sidebar',
  'open-register',
  'open-login',
  'open-admin',
  'open-orders',
  'logout',
  'open-cart'
])

const menuOpen = ref(false)
const scrolled = ref(false)

const toggleMenu = () => menuOpen.value = !menuOpen.value
const closeMenu = () => menuOpen.value = false

const abrirRegistro = () => {
  closeMenu()
  emit('open-register')
}

const abrirLogin = () => {
  closeMenu()
  emit('open-login')
}

const cerrarSesion = () => {
  closeMenu()
  emit('logout')
}

const handleScroll = () => {
  scrolled.value = window.scrollY > 20
}

onMounted(() => window.addEventListener('scroll', handleScroll))
onUnmounted(() => window.removeEventListener('scroll', handleScroll))

const navLinks = [
  { label: 'Colección', href: '#coleccion' },
  { label: 'Nosotros', href: '#nosotros' },
  { label: 'Sostenibilidad', href: '#sostenibilidad' },
  { label: 'Contacto', href: '#contacto' }
]

defineProps({
  usuario: {
    type: Object,
    default: null
  },
  cartCount: {
    type: Number,
    default: 0 }
})

const abrirPanelAdministrador = () => {
  closeMenu()
  emit('open-admin')
}

const abrirOrdenes = () => {
  closeMenu()
  emit('open-orders')
}

</script>

<template>
  <header :class="['ecora-header', { 'ecora-header--scrolled': scrolled }]">
    <div class="ecora-header__inner">

      <!-- MARCA -->
      <a href="#" class="ecora-brand" @click="closeMenu">
        <img src="/favicon.png" alt="Ecora" class="ecora-brand__logo">
        <span class="ecora-brand__name">ECORA</span>
      </a>

      <!-- NAV DESKTOP -->
      <nav class="ecora-nav" aria-label="Navegación principal">
        <a
          v-for="link in navLinks"
          :key="link.href"
          :href="link.href"
          class="ecora-nav__link"
        >
          {{ link.label }}
        </a>
      </nav>

      <!-- ACCIONES -->
      <div class="ecora-actions">
        <div
          v-if="!usuario"
          class="ecora-actions__auth"
        >
          <button
            class="ecora-actions__login"
            type="button"
            @click="abrirLogin"
          >
            Iniciar sesión
          </button>

          <button
            class="ecora-actions__register"
            type="button"
            @click="abrirRegistro"
          >
            Registrarse
          </button>
        </div>

        <div v-else class="ecora-actions__user">
        <span class="ecora-actions__user-name">
          Hola, {{ usuario.nombre }}
        </span>

        <button
          v-if="usuario.rol === 'cliente'"
          class="ecora-actions__orders"
          type="button"
          @click="abrirOrdenes"
        >
          Mis pedidos
        </button>

        <button
          v-if="usuario.rol === 'administrador'"
          class="ecora-actions__admin"
          type="button"
          @click="abrirPanelAdministrador"
        >
          Panel administrativo
        </button>

        <button
          class="ecora-actions__logout"
          type="button"
          @click="cerrarSesion"
        >
          Cerrar sesión
        </button>
      </div>

        <div>
          <button class="ecora-actions__cart" @click="emit('open-cart')" aria-label="Carrito">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M6 2L3 6v14a2 2 0 002 2h14a2 2 0 002-2V6l-3-4z" />
            <line x1="3" y1="6" x2="21" y2="6" />
            <path d="M16 10a4 4 0 01-8 0" />
          </svg>
            <span class="ecora-actions__cart-count">{{ cartCount }}</span>
          </button>
        </div>

        <!-- HAMBURGUESA MOBILE -->
        <button
          class="ecora-hamburger"
          :class="{ 'ecora-hamburger--open': menuOpen }"
          @click="toggleMenu"
          aria-label="Abrir menú"
          :aria-expanded="menuOpen"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>
      </div>
    </div>

    <!-- NAV MOBILE -->
    <Transition name="mobile-menu">
      <nav v-if="menuOpen" class="ecora-mobile-nav" aria-label="Menú móvil">
        <a
          v-for="link in navLinks"
          :key="link.href"
          :href="link.href"
          class="ecora-mobile-nav__link"
          @click="closeMenu"
        >
          {{ link.label }}
        </a>
        <div v-if="!usuario" class="ecora-mobile-nav__auth">
          <button
            class="ecora-mobile-nav__login"
            type="button"
            @click="abrirLogin"
          >
            Iniciar sesión
          </button>

          <button
            class="ecora-mobile-nav__register"
            type="button"
            @click="abrirRegistro"
          >
            Registrarse
          </button>
        </div>
        
         <div v-else class="ecora-mobile-nav__user">
          <p>
            Hola, {{ usuario.nombre }}
          </p>

          <button
            v-if="usuario.rol === 'cliente'"
            class="ecora-mobile-nav__orders"
            type="button"
            @click="abrirOrdenes"
          >
            Mis pedidos
          </button>

          <button
            v-if="usuario.rol === 'administrador'"
            class="ecora-mobile-nav__admin"
            type="button"
            @click="abrirPanelAdministrador"
          >
            Panel administrativo
          </button>

          <button
            class="ecora-mobile-nav__logout"
            type="button"
            @click="cerrarSesion"
          >
            Cerrar sesión
          </button>
        </div>  
      </nav>
    </Transition>
  </header>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,500;1,300;1,400&family=Jost:wght@300;400;500&display=swap');


/* ── Header base ── */
.ecora-header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--ecora-cream);
  border-bottom: 1px solid transparent;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  font-family: 'Jost', sans-serif;
}

.ecora-header--scrolled {
  border-bottom-color: var(--ecora-sand);
  box-shadow: 0 2px 16px rgba(11, 11, 9, 0.06);
}

.ecora-header__inner {
  max-width: 1280px;
  margin: 0 auto;
  padding: 0 2rem;
  height: 64px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 2rem;
}

/* ── Marca ── */
.ecora-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  text-decoration: none;
  flex-shrink: 0;
}

.ecora-brand__logo {
  width: 32px;
  height: 32px;
  object-fit: contain;
}

.ecora-brand__name {
  font-family: 'Cormorant Garamond', serif;
  font-size: 20px;
  font-weight: 400;
  letter-spacing: 0.2em;
  color: var(--ecora-black);
}

/* ── Nav desktop ── */
.ecora-nav {
  display: flex;
  align-items: center;
  gap: 2.5rem;
}

.ecora-nav__link {
  font-size: 11px;
  font-weight: 400;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  text-decoration: none;
  position: relative;
  padding-bottom: 2px;
  transition: color 0.2s;
}

.ecora-nav__link::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0;
  height: 1px;
  background: var(--ecora-stone);
  transition: width 0.25s ease;
}

.ecora-nav__link:hover {
  color: var(--ecora-black);
}

.ecora-nav__link:hover::after {
  width: 100%;
}

/* ── Acciones ── */
.ecora-actions {
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-shrink: 0;
}

.ecora-actions__register {
  padding: 8px 16px;
  border: 1px solid var(--ecora-black);
  background: transparent;
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.ecora-actions__register:hover {
  background: var(--ecora-black);
  color: var(--ecora-cream);
}

.ecora-mobile-nav__register {
  width: 100%;
  margin-top: 16px;
  padding: 12px;
  border: 1px solid var(--ecora-black);
  background: var(--ecora-black);
  color: var(--ecora-cream);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
}

.ecora-actions__cart {
  position: relative;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--ecora-stone);
  padding: 4px;
  display: flex;
  align-items: center;
  transition: color 0.2s;
}

.ecora-actions__cart:hover {
  color: var(--ecora-black);
}

.ecora-actions__cart-count {
  position: absolute;
  top: -4px;
  right: -6px;
  background: var(--ecora-stone);
  color: var(--ecora-cream);
  font-size: 9px;
  font-weight: 500;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  letter-spacing: 0;
}

/* ── Hamburguesa ── */
.ecora-hamburger {
  display: none;
  flex-direction: column;
  justify-content: center;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
  width: 28px;
}

.ecora-hamburger span {
  display: block;
  height: 1px;
  background: var(--ecora-stone);
  transition: transform 0.3s ease, opacity 0.3s ease, width 0.3s ease;
  transform-origin: center;
}

.ecora-hamburger span:nth-child(3) {
  width: 65%;
}

.ecora-hamburger--open span:nth-child(1) {
  transform: translateY(6px) rotate(45deg);
}

.ecora-hamburger--open span:nth-child(2) {
  opacity: 0;
  transform: scaleX(0);
}

.ecora-hamburger--open span:nth-child(3) {
  width: 100%;
  transform: translateY(-6px) rotate(-45deg);
}

/* ── Nav mobile ── */
.ecora-mobile-nav {
  display: flex;
  flex-direction: column;
  background: var(--ecora-cream);
  border-top: 1px solid var(--ecora-sand);
  padding: 1.25rem 2rem 1.5rem;
  gap: 0;
}

.ecora-mobile-nav__link {
  font-size: 11px;
  font-weight: 400;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  text-decoration: none;
  padding: 0.875rem 0;
  border-bottom: 1px solid rgba(174, 168, 151, 0.3);
  transition: color 0.2s;
}

.ecora-mobile-nav__link:last-child {
  border-bottom: none;
}

.ecora-mobile-nav__link:hover {
  color: #0B0B09;
}

/* ── Transición menú mobile ── */
.mobile-menu-enter-active,
.mobile-menu-leave-active {
  transition: opacity 0.25s ease, transform 0.25s ease;
}

.mobile-menu-enter-from,
.mobile-menu-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

.ecora-actions__auth {
  display: flex;
  align-items: center;
  gap: 10px;
}

.ecora-actions__login {
  padding: 8px 12px;
  border: none;
  background: transparent;
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
}

.ecora-actions__login:hover {
  text-decoration: underline;
}

.ecora-actions__user {
  color: var(--ecora-black);
  font-size: 13px;
  font-weight: 500;
}

.ecora-mobile-nav__auth {
  width: 100%;
  margin-top: 16px;
}

.ecora-mobile-nav__login {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  border: 1px solid var(--ecora-black);
  background: transparent;
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
}

.ecora-mobile-nav__user {
  width: 100%;
  margin-top: 16px;
  text-align: center;
  font-weight: 500;
}

.ecora-actions__user {
  display: flex;
  align-items: center;
  gap: 12px;
}

.ecora-actions__user-name {
  color: var(--ecora-black);
  font-size: 13px;
  font-weight: 500;
}

.ecora-actions__logout {
  padding: 8px 12px;
  border: 1px solid var(--ecora-black);
  background: transparent;
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.ecora-actions__logout:hover {
  background: var(--ecora-black);
  color: var(--ecora-cream);
}

.ecora-mobile-nav__user {
  width: 100%;
  margin-top: 16px;
  text-align: center;
}

.ecora-mobile-nav__user p {
  margin-bottom: 12px;
  font-weight: 500;
}

.ecora-mobile-nav__logout {
  width: 100%;
  padding: 12px;
  border: 1px solid var(--ecora-black);
  background: transparent;
  color: var(--ecora-black);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
}

.ecora-mobile-nav__logout:hover {
  background: var(--ecora-black);
  color: var(--ecora-cream);
}
/* ── Responsive ── */
@media (max-width: 768px) {
  .ecora-nav {
    display: none;
  }

  .ecora-actions__register {
  display: none;
  }

  .ecora-hamburger {
    display: flex;
  }

  .ecora-header__inner {
    padding: 0 1.25rem;
  }

  .ecora-actions__auth,
  .ecora-actions__user {
    display: none;
  }
}

.ecora-actions__orders {
  padding: 8px 12px;
  border: 1px solid var(--ecora-warm);
  background: transparent;
  color: var(--ecora-warm);
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.ecora-actions__orders:hover {
  background: var(--ecora-warm);
  color: var(--ecora-cream);
}

.ecora-mobile-nav__orders {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  border: 1px solid var(--ecora-warm);
  background: transparent;
  color: var(--ecora-warm);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
}

.ecora-mobile-nav__orders:hover {
  background: var(--ecora-warm);
  color: var(--ecora-cream);
}

.ecora-actions__admin {
  padding: 8px 12px;
  border: 1px solid var(--ecora-warm);
  background: var(--ecora-warm);
  color: var(--ecora-cream);
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  cursor: pointer;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.ecora-actions__admin:hover {
  background: transparent;
  color: var(--ecora-warm);
}

.ecora-mobile-nav__admin {
  width: 100%;
  margin-bottom: 10px;
  padding: 12px;
  border: 1px solid var(--ecora-warm);
  background: var(--ecora-warm);
  color: var(--ecora-cream);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  cursor: pointer;
}

</style>