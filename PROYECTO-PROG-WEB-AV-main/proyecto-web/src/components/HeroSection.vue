<script setup>
const emit = defineEmits(['open-collection'])
const Navlink= {href: '#coleccion'}
</script>

<template>
  <section class="hero">
    <div class="hero__content">

      <!-- EYEBROW -->
      <p class="hero__eyebrow">Moda Sostenible · Tela Reciclada</p>

      <!-- TÍTULO -->
      <h1 class="hero__title">
        La moda también<br>
        <em>puede renacer.</em>
      </h1>

      <!-- DESCRIPCIÓN -->
      <p class="hero__description">
        Cada prenda de Ecora nace de tela reciclada y producción
        responsable. Vestir consciente nunca había sido tan hermoso.
      </p>

      <!-- BOTONES -->
      <div class="hero__actions">
        <button @click="emit('open-collection')" class="hero__btn hero__btn--primary">
          Ver Colección
        </button>
        <a href="#nosotros" class="hero__btn hero__btn--secondary">
          Nuestra Historia
        </a>
      </div>

      <!-- ESTADÍSTICAS -->
      <div class="hero__stats">
        <div class="hero__stat">
          <span class="hero__stat-number">100%</span>
          <span class="hero__stat-label">Tela Reciclada</span>
        </div>
        <div class="hero__stat">
          <span class="hero__stat-number">0</span>
          <span class="hero__stat-label">Residuos Tóxicos</span>
        </div>
        <div class="hero__stat">
          <span class="hero__stat-number">+2K</span>
          <span class="hero__stat-label">Prendas Renovadas</span>
        </div>
      </div>

    </div>

    <!-- IMAGEN -->
    <div class="hero__image-wrap">
      <img src="/hero.jpg" alt="Colección Ecora — ropa sostenible, imagen tomada de Pinterest @BeautyHarbour"
        class="hero__image">
    </div>

    <a href="#coleccion" class="hero__scroll">
      <span class="hero__scroll-icon">↓</span>
      <span class="hero__scroll-text">Colección 2026</span>
    </a>
  </section>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;1,300;1,400&family=Jost:wght@300;400;500&display=swap');

.hero {
  display: grid;
  grid-template-columns: 1fr 1fr;
  background: #F5F3EE;
  min-height: calc(100vh - 64px);
  max-height: 680px;
  overflow: hidden;
  position: relative;
}

/* ── Contenido izquierdo ── */
.hero__content {
  display: flex;
  flex-direction: column;
  gap: 1.75rem;
  padding: 2rem 4rem 2rem 6rem;
  justify-content: flex-start;
}

.hero__eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #716C59;
}

.hero__title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(3rem, 5vw, 4.5rem);
  font-weight: 400;
  line-height: 1.1;
  color: #0B0B09;
}

.hero__title em {
  font-style: italic;
  font-weight: 300;
}

.hero__description {
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  font-weight: 300;
  line-height: 1.75;
  color: #716C59;
  max-width: 380px;
}

/* ── Botones ── */
.hero__actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.hero__btn {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  padding: 1rem 1.75rem;
  text-decoration: none;
  transition: background 0.25s ease, color 0.25s ease;
  cursor: pointer;
  display: inline-block;
}

.hero__btn--primary {
  background: #716C59;
  color: #F5F3EE;
  border: 1px solid #716C59;
}

.hero__btn--primary:hover {
  background: #0B0B09;
  border-color: #0B0B09;
}

.hero__btn--secondary {
  background: transparent;
  color: #0B0B09;
  border: 1px solid #0B0B09;
}

.hero__btn--secondary:hover {
  background: #0B0B09;
  color: #F5F3EE;
}

/* ── Estadísticas ── */
.hero__stats {
  display: flex;
  gap: 3rem;
  padding-top: 1.5rem;
  border-top: 1px solid #AEA897;
  margin-top: 0.5rem;
}

.hero__stat {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.hero__stat-number {
  font-family: 'Cormorant Garamond', serif;
  font-size: 2rem;
  font-weight: 400;
  color: #0B0B09;
  line-height: 1;
}

.hero__stat-label {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: #716C59;
}

/* ── Imagen derecha ── */
.hero__image-wrap {
  position: relative;
  overflow: hidden;
  height: 100%;
}

.hero__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  display: block;
}

/* Badge colección 2026*/
.hero__scroll {
  position: absolute;
  bottom: 2rem;
  right: 2rem;
  display: flex;
  align-items: center;
  gap: 0.75rem;
  background: #F5F3EE;
  padding: 0.625rem 1.25rem;
  z-index: 10;
  text-decoration: none;
  cursor: pointer;
}

.hero__scroll-icon {
  font-size: 14px;
  color: #716C59;
  animation: bounce 2s infinite;
}

.hero__scroll-text {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: #0B0B09;
}

@keyframes bounce {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(4px);
  }
}

/* ── Responsive ── */
@media (max-width: 900px) {
  .hero {
    grid-template-columns: 1fr;
    height: auto;
    max-height: none;
    overflow: visible;
  }

  .hero__content {
    padding: 3rem 1.25rem 2.5rem;
    order: 1;
  }

  .hero__imagen-wrap {
    height: 55vw;
    min-height: 280px;
    order: 0;
  }


  .hero__scroll {
    position: absolute;
    bottom: 1rem;
    right: 1rem;
    z-index: 10;
  }
}

@media (max-width: 480px) {
  .hero__acciones {
    flex-direction: column;
    gap: 0.75rem;
  }

  .hero__btn {
    text-align: center;
    width: 100%;
  }

  .hero__stats {
    gap: 1.5rem;
    flex-wrap: wrap;
  }
}
</style>