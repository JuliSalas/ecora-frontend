<script setup>
const teamMembers = [
  {
    name: 'Diego Calvo Sanabria',
    age: '20 años',
    role: 'Lector de cómics',
    image: '/img_Diego.jpg',
  },
  {
    name: 'Selena Quirós Serrano',
    age: '30 años',
    role: 'Oyente de música',
    image: '/img_Selena.jpeg',
  },
  {
    name: 'Carlos Fuentes Santos',
    age: '24 años',
    role: 'Jugador de fútbol',
    image: '/img_Carlos.jpeg',
  },
]

const teamQuote =
  'Somos tres personas con pasiones distintas unidas por una misma convicción: la moda puede —y debe— ser parte del cambio.'
</script>

<template>
  <section class="team-section" id="team">
    <div class="container">

      <div class="team-header">
        <h2 class="team-title">
          Quienes crean <span>Ecora</span>
        </h2>
      </div>

      <div class="team-grid">
        <article
          v-for="member in teamMembers"
          :key="member.name"
          class="member-card"
        >
          <div class="member-image-wrapper">
            <img
              :src="member.image"
              :alt="member.name"
              class="member-image"
            >
          </div>

          <h3 class="member-name">
            {{ member.name }}
          </h3>

          <p class="member-age">
            {{ member.age }}
          </p>

          <p class="member-role">
            {{ member.role }}
          </p>
        </article>
      </div>

      <div class="team-divider"></div>

      <blockquote class="team-quote">
        "{{ teamQuote }}"
      </blockquote>

    </div>
  </section>
</template>

<style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@300;400;500;600&family=Jost:wght@300;400;500&display=swap');

.team-section {
  background: var(--ecora-cream);
  padding: 7rem 0;
}

.team-header {
  text-align: center;
  margin-bottom: 4rem;
}

.team-title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(3rem, 5vw, 4.5rem);
  font-weight: 400;
  line-height: 1;
  margin: 0;
  color: var(--ecora-black);
}

.team-title span {
  font-style: italic;
  font-weight: 400;
}

.team-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 6rem;
  max-width: 1000px;
  margin: 0 auto;
}

.member-card {
  text-align: center;
  min-width: 0;
}

.member-image-wrapper {
  margin-bottom: 1.75rem;
}

.member-image {
  width: 100%;
  aspect-ratio: 4 / 5;
  object-fit: cover;
  display: block;
}

.member-name {
  font-family: 'Cormorant Garamond', serif;
  font-size: 2rem;
  font-weight: 500;
  line-height: 1.15;
  margin: 0 0 0.5rem;
  color: var(--ecora-black);
}

.member-age {
  font-family: 'Jost', sans-serif;
  text-transform: uppercase;
  letter-spacing: 0.15em;
  font-size: 0.95rem;
  color: var(--ecora-stone);
  margin: 0 0 0.75rem;
}

.member-role {
  font-family: 'Jost', sans-serif;
  font-size: 1.1rem;
  color: var(--ecora-stone);
  margin: 0;
}

.team-divider {
  max-width: 720px;
  margin: 6rem auto 4rem;
  border-top: 1px solid var(--ecora-sand);
}

.team-quote {
  max-width: 850px;
  margin: 0 auto;

  text-align: center;

  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(1.25rem, 2vw, 1.75rem);
  font-style: italic;
  font-weight: 200;
  line-height: 1.4;
}

@media (max-width: 768px) {
  .team-section {
    padding: 5rem 0;
  }

  .team-grid {
    grid-template-columns: 1fr;
    gap: 3rem;
  }

  .team-divider {
    margin: 4rem auto 2.5rem;
  }
}
</style>