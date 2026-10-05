<script setup>
// TODO: cuando existan las rutas de cada categoría/página, cambiar
// los <a href="#"> por <router-link :to="...">.
const currentYear = new Date().getFullYear()

const shopLinks = [
  { label: 'Nueva colección', tipo: 'Todos', href: '#coleccion' },
  { label: 'Camisetas', tipo: 'Camisetas', href: '#coleccion' },
  { label: 'Faldas', tipo: 'Faldas', href: '#coleccion' },
  { label: 'Chaquetas', tipo: 'Jacket', href: '#coleccion' },
  { label: 'Pantalones', tipo: 'Pantalones', href: '#coleccion' },
]

const companyLinks = [
  { label: 'Nuestra historia', href: '#team' },
  { label: 'Proceso productivo', href: '#sostenibilidad' },
  { label: 'Contacto', href: '#contacto' },
]

const socials = [
  {
    label: 'Instagram',
    href: '#',
    icon: `<rect x="3" y="3" width="18" height="18" rx="5" fill="none" stroke="currentColor" stroke-width="1.6"/>
          <circle cx="12" cy="12" r="4" fill="none" stroke="currentColor" stroke-width="1.6"/>
          <circle cx="17.3" cy="6.7" r="0.9" fill="currentColor"/>`,
  },
  {
    label: 'Facebook',
    href: '#',
    icon: `<path d="M15 4.5h-2.2a3 3 0 0 0-3 3V10H7.5v3H9.8v6.5h3V13h2.4l.4-3h-2.8V7.8c0-.7.5-1.3 1.3-1.3H15z" fill="currentColor"/>`,
  },
  {
    label: 'Twitter',
    href: '#',
    icon: `<path d="M22 5.9c-.7.3-1.4.5-2.2.6.8-.5 1.4-1.2 1.7-2.1-.7.5-1.6.8-2.4 1a3.8 3.8 0 0 0-6.5 3.5A10.8 10.8 0 0 1 4.9 4.9a3.8 3.8 0 0 0 1.2 5.1c-.6 0-1.2-.2-1.7-.5v.1c0 1.9 1.3 3.4 3.1 3.8-.6.1-1.1.2-1.7.1a3.8 3.8 0 0 0 3.6 2.6A7.6 7.6 0 0 1 3 17.5a10.7 10.7 0 0 0 5.8 1.7c7 0 10.8-5.9 10.8-11v-.5c.7-.5 1.4-1.2 1.9-2z" fill="none" stroke="currentColor" stroke-width="1.4" stroke-linecap="round" stroke-linejoin="round"/>`,
  },
]
const emit = defineEmits(['filtrar-coleccion'])

const irAColeccion = (tipo) => {
  emit('filtrar-coleccion', tipo)
}
</script>
<template>
  <footer class="app-footer">
    <div class="container">
      <div class="row gy-5">
        <div class="col-12 col-lg-5">
          <div class="footer-brand d-flex align-items-center gap-3 mb-3">
            <span class="footer-logo" aria-hidden="true"><img src="/favicon.png" alt="Ecora"
                class="ecora-brand__logo"></span>
            <span class="footer-brand-name">Ecora</span>
          </div>

          <p class="footer-tagline">
            &quot;La moda también puede renacer.&quot; Creamos prendas únicas
            a partir de tela reciclada, con diseño, conciencia y propósito.
          </p>

          <div class="footer-social d-flex gap-2">
            <a v-for="social in socials" :key="social.label" :href="social.href" class="footer-social-link"
              :aria-label="social.label">
              <svg viewBox="0 0 24 24" v-html="social.icon"></svg>
            </a>
          </div>
        </div>

        <div class="col-6 col-lg-3">
          <h3 class="footer-heading">Tienda</h3>
          <ul class="footer-links list-unstyled">
            <li v-for="link in shopLinks" :key="link.label">
              <a href="#coleccion" @click.prevent="irAColeccion(link.tipo, link.href)">
                {{ link.label }}
              </a>
            </li>
          </ul>
        </div>
        <div class="col-6 col-lg-3 offset-lg-1">
          <h3 class="footer-heading">Empresa</h3>
          <ul class="footer-links list-unstyled">

            <li v-for="link in companyLinks" :key="link.href">
              <a :href="link.href">{{ link.label }}</a>
            </li>
          </ul>
        </div>
      </div>

      <hr class="footer-divider" />

      <div
        class="footer-bottom d-flex flex-column flex-sm-row justify-content-sm-between gap-2 text-center text-sm-start">
        <p class="mb-0">© {{ currentYear }} Ecora. Todos los derechos reservados.</p>
        <p class="mb-0">Hecho con conciencia · Costa Rica</p>
      </div>
    </div>
  </footer>
</template>


<style scoped>
.app-footer {
  background-color: var(--ecora-black);
  color: var(--ecora-sand);
  padding: 4.5rem 0 2rem;
}

.footer-logo {
  display: inline-block;
  width: 48px;
  height: 48px;
  flex-shrink: 0;
  border-radius: 4px;
  background-color: var(--ecora-sand);
}

.footer-brand-name {
  font-family: var(--ecora-font-serif, 'Playfair Display', Georgia, serif);
  font-weight: 700;
  font-size: 1.5rem;
  color: var(--ecora-cream);
}

.footer-tagline {
  max-width: 360px;
  font-size: 0.95rem;
  line-height: 1.6;
  color: var(--ecora-sand);
  margin-bottom: 1.5rem;
}

.footer-social-link {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  border: 1px solid rgba(245, 243, 238, 0.25);
  border-radius: 6px;
  color: var(--ecora-cream);
  text-decoration: none;
  transition: border-color 0.2s ease, color 0.2s ease;
}

.footer-social-link svg {
  width: 18px;
  height: 18px;
}

.footer-social-link:hover,
.footer-social-link:focus-visible {
  border-color: var(--ecora-sand);
  color: var(--ecora-sand);
}

.footer-heading {
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--ecora-cream);
  margin-bottom: 1.25rem;
}

.footer-links li+li {
  margin-top: 0.85rem;
}

.footer-links a {
  color: var(--ecora-sand);
  text-decoration: none;
  font-size: 0.95rem;
  position: relative;
  transition: color 0.2s ease;
}

.footer-links a:hover,
.footer-links a:focus-visible {
  color: var(--ecora-cream);
}

.footer-divider {
  border: none;
  border-top: 1px solid rgba(245, 243, 238, 0.12);
  margin: 3rem 0 1.5rem;
  opacity: 1;
}

.footer-bottom {
  font-size: 0.85rem;
  color: var(--ecora-gray);
}
</style>