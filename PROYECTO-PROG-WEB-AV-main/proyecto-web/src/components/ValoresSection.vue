<script setup>

const values = [
  {
    icon: '🌱',
    title: 'Sostenibilidad',
    description: 'Cada decisión considera su impacto en el planeta.',
  },
  {
    icon: '🤝',
    title: 'Responsabilidad',
    description: 'Productos alineados con una visión ética y consciente.',
  },
  {
    icon: '✦',
    title: 'Estilo',
    description: 'La moda ecológica también puede ser moderna y versátil.',
  },
  {
    icon: '◎',
    title: 'Transparencia',
    description: 'Comunicamos con claridad el origen de cada prenda.',
  },
  {
    icon: '↻',
    title: 'Renovación',
    description: 'La moda puede renacer con una nueva vida.',
  },
]
</script>

<template>
  <section class="values-section">
    <div class="container">
      <div class="values-header text-center">
        <p class="values-eyebrow">Lo que nos guía</p>
        <h2 class="values-title">Nuestros Valores</h2>
      </div>

      <div class="values-grid">
        <article
          v-for="value in values"
          :key="value.title"
          class="value-card"
        >
          <span class="value-icon" aria-hidden="true">{{ value.icon }}</span>
          <h3 class="value-title">{{ value.title }}</h3>
          <p class="value-description">{{ value.description }}</p>
        </article>
      </div>
    </div>
  </section>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;1,300;1,400&family=Jost:wght@300;400;500&display=swap');

.values-section {
  background: var(--ecora-cream);
  padding: 7rem 0;
}

.values-header {
  text-align: center;
  margin-bottom: 4rem;
}

.values-eyebrow {
  font-family: 'Jost', sans-serif;
  font-size: 0.75rem;
  letter-spacing: 0.35em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  margin-bottom: 1rem;
}

.values-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(3rem, 5vw, 4rem);
  font-weight: 400;
  color: var(--ecora-black);
  margin: 0;
}

/* GRID */

.values-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);

  border-top: 1px solid var(--ecora-sand);
  border-left: 1px solid var(--ecora-sand);
}

/* CARD */

.value-card {
  min-height: 190px;

  padding: 2.5rem 2rem;

  border-right: 1px solid var(--ecora-sand);
  border-bottom: 1px solid var(--ecora-cream);

  background: transparent;

  transition: all 0.3s ease;
}

.value-card:hover {
  background: rgba(0, 0, 0, 0.02);
}

/* ICON */

.value-icon {
  display: block;
  font-size: 1.5rem;
  margin-bottom: 1.75rem;
  line-height: 1;
}

/* TITLE */

.value-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: 2rem;
  font-weight: 600;
  line-height: 1.1;

  margin-bottom: 1rem;

  color: var(--ecora-black);
}

/* DESCRIPTION */

.value-description {
  font-family: 'Jost', sans-serif;
  font-size: 0.95rem;
  line-height: 1.7;
  color: var(--ecora-stone);

  max-width: 220px;
}

/* TABLET */

@media (max-width: 1200px) {
  .values-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* MOBILE */

@media (max-width: 768px) {
  .values-section {
    padding: 5rem 0;
  }

  .values-grid {
    grid-template-columns: 1fr;
  }

  .value-card {
    min-height: auto;
    padding: 2rem;
  }

  .values-title {
    font-size: 2.75rem;
  }
}
</style>