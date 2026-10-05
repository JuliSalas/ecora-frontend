<template>
  <section class="nosotros" id="nosotros">
    <div class="nosotros__inner">

      <!-- COLUMNA IMAGEN -->
      <div class="nosotros__image-wrap">
        <img
          src="/nosotros.jpg"
          alt="Telas recicladas en taller de Ecora, fotografía tomada de @Greenlosophy en Pinterest"
          class="nosotros__image"
        >
        <div class="nosotros__badge">
          <span class="nosotros__badge-year">DESDE 2023</span>
          <p class="nosotros__badge-text">Moda que respeta el planeta</p>
        </div>
      </div>

      <!-- COLUMNA TEXTO -->
      <div class="nosotros__content">

        <span class="nosotros__eyebrow">Quiénes somos</span>

        <h2 class="nosotros__title">
          Una marca nacida<br>
          del <em>compromiso.</em>
        </h2>

        <p class="nosotros__description">
          Ecora nació de la convicción de que la industria de la moda puede
          —y debe— cambiar. Creamos prendas únicas a partir de tela reciclada,
          dándole una segunda vida a materiales que de otro modo terminarían
          en vertederos.
        </p>

        <!-- MISIÓN Y VISIÓN -->
        <div class="nosotros__mv">
          <div class="nosotros__mv-item">
            <h3 class="nosotros__mv-title">Misión</h3>
            <p class="nosotros__mv-text">
              Brindar una alternativa de moda sostenible con prendas ecológicas,
              promoviendo un consumo consciente sin sacrificar estilo ni calidad.
            </p>
          </div>
          <div class="nosotros__mv-item">
            <h3 class="nosotros__mv-title">Visión</h3>
            <p class="nosotros__mv-text">
              Convertirnos en marca referente de moda sostenible, reconocida por
              demostrar que la ropa puede tener una segunda vida sin perder diseño
              ni personalidad.
            </p>
          </div>
        </div>

      </div>
    </div>
  </section>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;1,300;1,400&family=Jost:wght@300;400;500&display=swap');

.nosotros {
  background: var(--ecora-cream);
  padding: 7rem 2rem;
}

.nosotros__inner {
  max-width: 1200px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 5rem;
  align-items: center;
}

/* ── Imagen ── */
.nosotros__image-wrap {
  position: relative;
}

.nosotros__image {
  width: 100%;
  height: 580px;
  object-fit: cover;
  display: block;
}

.nosotros__badge {
  position: absolute;
  bottom: 2rem;
  left: 2rem;
  background: var(--ecora-cream);
  padding: 1.25rem 1.5rem;
  max-width: 200px;
}

.nosotros__badge-year {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  display: block;
  margin-bottom: 0.5rem;
}

.nosotros__badge-text {
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  font-weight: 400;
  color: var(--ecora-black);
  line-height: 1.4;
}

/* ── Contenido ── */
.nosotros__content {
  display: flex;
  flex-direction: column;
  gap: 1.75rem;
}

.nosotros__eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.2em;
  text-transform: uppercase;
  color: var(--ecora-stone);
}

.nosotros__title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2.5rem, 4vw, 3.5rem);
  font-weight: 400;
  line-height: 1.15;
  color: var(--ecora-black);
}

.nosotros__title em {
  font-style: italic;
  font-weight: 300;
}

.nosotros__description {
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  font-weight: 300;
  line-height: 1.75;
  color: var(--ecora-stone);
  max-width: 480px;
}

/* ── Misión y Visión ── */
.nosotros__mv {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0;
  border-top: 1px solid var(--ecora-sand);
  padding-top: 1.75rem;
}

.nosotros__mv-item {
  padding-right: 2rem;
  border-left: 2px solid var(--ecora-sand);
  padding-left: 1.25rem;
}

.nosotros__mv-item:first-child {
  border-left: none;
  padding-left: 0;
  border-right: 1px solid var(--ecora-sand);
  padding-right: 2rem;
}

.nosotros__mv-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: 18px;
  font-weight: 400;
  color: var(--ecora-black);
  margin-bottom: 0.75rem;
}

.nosotros__mv-text {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  font-weight: 300;
  line-height: 1.7;
  color: var(--ecora-stone);
}

/* ── Responsive ── */
@media (max-width: 900px) {
  .nosotros__inner {
    grid-template-columns: 1fr;
    gap: 3rem;
  }

  .nosotros__image {
    height: 420px;
  }

  .nosotros__title {
    font-size: 2.25rem;
  }
}

@media (max-width: 480px) {
  .nosotros {
    padding: 4rem 1.25rem;
  }

  .nosotros__mv {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .nosotros__mv-item:first-child {
    border-right: none;
    border-bottom: 1px solid #AEA897;
    padding-bottom: 1.5rem;
  }

  .nosotros__mv-item {
    padding-right: 0;
  }
}
</style>