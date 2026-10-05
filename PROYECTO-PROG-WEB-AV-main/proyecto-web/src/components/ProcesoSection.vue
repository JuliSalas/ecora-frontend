<script setup>
const processSteps = [
  {
    number: '01',
    title: 'Recolección',
    description:
      'Recolectamos telas y prendas descartadas de talleres locales, donaciones y excedentes industriales.',
  },
  {
    number: '02',
    title: 'Selección',
    description:
      'Clasificamos cada material según calidad, textura y potencial creativo para asegurar durabilidad.',
  },
  {
    number: '03',
    title: 'Diseño',
    description:
      'Nuestros diseñadores crean piezas únicas respetando la naturaleza de cada tela recuperada.',
  },
  {
    number: '04',
    title: 'Confección',
    description:
      'Producción local y artesanal con procesos de bajo impacto ambiental y sin residuos tóxicos.',
  },
]

const metrics = [
  {
    value: '8.2 T',
    label: 'Tela recuperada',
  },
  {
    value: '93%',
    label: 'Residuo reducido',
  },
  {
    value: '1,400',
    label: 'Prendas únicas',
  },
]
</script>

<template>
  <section class="process-section" id="sostenibilidad">
    <div class="container">
      <div class="process-layout">

        <div class="process-content">
          <p class="process-eyebrow">
            Nuestro proceso
          </p>

          <h2 class="process-title">
            Del desecho a la
            <span>prenda perfecta.</span>
          </h2>

          <p class="process-description">
            Cada pieza de Ecora sigue un proceso cuidadoso que transforma
            materiales descartados en moda con propósito. Así funciona
            nuestra cadena de valor sostenible.
          </p>

          <div class="steps-list">
            <article
              v-for="step in processSteps"
              :key="step.number"
              class="step-item"
            >
              <span class="step-number">
                {{ step.number }}
              </span>

              <div class="step-content">
                <h3 class="step-title">
                  {{ step.title }}
                </h3>

                <p class="step-description">
                  {{ step.description }}
                </p>
              </div>
            </article>
          </div>
        </div>

        <div class="process-visual">
          <div class="process-image-wrapper">
            <img
              src="/process-fabric.jpg"
              alt="Tela reciclada, imagen tomada de Pinterest @Ricardo Almeida"
              class="process-image"
            >
          </div>

          <div class="metrics-grid">
            <div
              v-for="metric in metrics"
              :key="metric.label"
              class="metric-item"
            >
              <h3 class="metric-value">
                {{ metric.value }}
              </h3>

              <p class="metric-label">
                {{ metric.label }}
              </p>
            </div>
          </div>
        </div>

      </div>
    </div>
  </section>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500;600&family=Jost:wght@300;400;500&display=swap');

.process-section {
  background: var(--ecora-black);
  padding: 4.5rem 0;
  color: var(--ecora-cream);
  position: relative;
}

.process-layout {
  display: grid;
  grid-template-columns: 1fr 0.95fr;
  gap: 4rem;
  align-items: start;
}

/* HEADER */

.process-eyebrow {
  font-family: 'Jost', sans-serif;
  text-transform: uppercase;
  letter-spacing: 0.3em;
  font-size: 0.75rem;
  color: rgba(255,255,255,0.7);
  margin-bottom: 1.5rem;
}

.process-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2.8rem, 4vw, 4.2rem);
  line-height: 0.95;
  font-weight: 400;
  margin: 0 0 2rem;
}

.process-title span {
  display: block;
  font-style: italic;
}

.process-description {
  max-width: 650px;
  font-family: 'Jost', sans-serif;
  font-size: 1.1rem;
  line-height: 1.8;
  color: rgba(255,255,255,0.75);
  margin-bottom: 2rem;
}

/* STEPS */

.steps-list {
  max-width: 600px;
  position: relative;
}

.step-item {
  display: grid;
  grid-template-columns: 60px 1fr;
  gap: 1.5rem;
  padding: 2rem 0;
  border-top: 1px solid rgba(255,255,255,0.08);
}

.step-number {
  font-family: 'Cormorant Garamond', serif;
  font-size: 2rem;
  color: var(--ecora-warm);
}

.step-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: 2rem;
  font-weight: 600;
  margin: 0 0 0.75rem;
}

.step-description {
  font-family: 'Jost', sans-serif;
  color: rgba(255,255,255,0.72);
  line-height: 1.8;
  margin: 0;
}

/* IMAGE */

.process-image-wrapper {
  overflow: hidden;
}

.process-image {
  display: block;
  width: 100%;
  height: 320px;
  object-fit: cover;
}

/* METRICS */

.metrics-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);

  margin-top: 2rem;
}

.metric-item {
  text-align: center;
  padding: 1.5rem;

  border-left: 1px solid rgba(255,255,255,0.08);
}

.metric-item:first-child {
  border-left: none;
}

.metric-value {
  font-family: 'Cormorant Garamond', serif;
  font-size: 3rem;
  font-weight: 600;
  margin: 0 0 0.5rem;
}

.metric-label {
  font-family: 'Jost', sans-serif;
  text-transform: uppercase;
  letter-spacing: 0.15em;
  font-size: 0.75rem;
  color: rgba(255,255,255,0.55);
  margin: 0;
}

/* RESPONSIVE */

@media (max-width: 1100px) {
  .process-layout {
    grid-template-columns: 1fr;
  }

  .process-visual {
    order: -1;
  }
}

@media (max-width: 768px) {
  .process-section {
    padding: 5rem 0;
  }

  .process-title {
    font-size: 3rem;
  }

  .metrics-grid {
    grid-template-columns: 1fr;
  }

  .metric-item {
    border-left: none;
    border-top: 1px solid rgba(255,255,255,0.08);
  }

  .step-item {
    grid-template-columns: 1fr;
    gap: 1rem;
  }
}
</style>