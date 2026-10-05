<script setup>
import { ref } from 'vue'

const email = ref('')
const enviado = ref(false)
const error = ref('')
const cargando = ref(false)

const validateEmail = (email) => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

const subscribe = async () => {
  error.value = ''

  if (!email.value.trim()) {
    error.value = 'Por favor ingresá tu correo electrónico.'
    return
  }

  if (!validateEmail(email.value)) {
    error.value = 'El correo ingresado no es válido.'
    return
  }

  try {
    cargando.value = true

    const response = await fetch('http://localhost:3000/api/email', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ email: email.value })
    })

    const data = await response.json()
    console.log('Respuesta del servidor:', data)

    if (!response.ok) {
      error.value = data.message || 'Hubo un error al enviar el correo.'
      return
    }

    if (!data.success) {
      error.value = data.message || 'No se pudo enviar el correo.'
      return
    }

    enviado.value = true
    email.value = ''

  } catch (err) {
    error.value = 'No se pudo conectar con el servidor. Intentá más tarde.'
  } finally {
    cargando.value = false
  }
}
</script>

<template>
  <section class="contact" id="contacto">
    <div class="contact__inner">

      <!-- Título -->
      <h2 class="contact__title">
        Únete al movimiento<br>
        de moda <em>consciente.</em>
      </h2>

      <!-- Descripción -->
      <p class="contact__description">
        Suscríbete y recibe acceso anticipado a nuevas colecciones,
        guías de cuidado sostenible y descuentos exclusivos para la comunidad.
      </p>

      <!-- Formulario -->
      <div v-if="!enviado" class="contact__form">
        <div class="contact__layout">
          <input v-model="email" type="email" placeholder="tu@correo.com" class="contact__input"
            :class="{ 'contact__input--error': error }" aria-label="Correo electrónico" @keyup.enter="subscribe">
          <button class="contact__btn" @click="subscribe" :disabled="cargando">
            {{ cargando ? 'Enviando...' : 'Suscribir →' }}
          </button>
        </div>
        <!-- Error -->
        <p v-if="error" class="contact__error">{{ error }}</p>

        <!-- Nota -->
        <p class="contact__note">
          Sin spam. Solo contenido que vale la pena.
          Puedes darte de baja en cualquier momento.
        </p>
      </div>

      <!-- Mensaje de éxito -->
      <div v-else class="contact__subscribed">
        <p class="contact__subscribed-text">
          ¡Gracias por unirte! Pronto recibirás noticias de Ecora.
        </p>
        <button class="contact__subscribed-btn" @click="enviado = false">
          Suscribir otro correo
        </button>
      </div>

    </div>
  </section>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;1,300;1,400&family=Jost:wght@300;400;500&display=swap');

.contact {
  background: #F5F3EE;
  padding: 7rem 2rem;
}

.contact__inner {
  max-width: 700px;
  margin: 0 auto;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
}

/* ── Título ── */
.contact__title {
  font-family: 'Cormorant Garamond', serif;
  font-size: clamp(2.5rem, 5vw, 4rem);
  font-weight: 400;
  line-height: 1.15;
  color: var(--ecora-black);
}

.contact__title em {
  font-style: italic;
  font-weight: 300;
}

/* ── Descripción ── */
.contact__description {
  font-family: 'Jost', sans-serif;
  font-size: 15px;
  font-weight: 300;
  line-height: 1.75;
  color: var(--ecora-sand);
  max-width: 520px;
}

/* ── Formulario ── */
.contact__form {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  margin-top: 0.5rem;
}

.contact__layout {
  display: flex;
  width: 100%;
  max-width: 580px;
  gap: 0;
}

.contact__input {
  flex: 1;
  padding: 1rem 1.25rem;
  border: 1px solid var(--ecora-sand);
  border-right: none;
  background: var(--ecora-cream);
  font-family: 'Jost', sans-serif;
  font-size: 14px;
  color: var(--ecora-black);
  outline: none;
  transition: border-color 0.2s;
}

.contact__input::placeholder {
  color: var(--ecora-sand);
}

.contact__input:focus {
  border-color: var(--ecora-stone);
}

.contact__input--error {
  border-color: var(--ecora-error);
}

.contact__btn {
  padding: 1rem 1.75rem;
  background: var(--ecora-stone);
  color: var(--ecora-cream);
  border: 1px solid var(--ecora-stone);
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.2s;
}

.contact__btn:hover {
  background: var(--ecora-black);
  border-color: var(--ecora-black);
}

/* ── Error ── */
.contact__error {
  font-family: 'Jost', sans-serif;
  font-size: 13px;
  color: var(--ecora-error);
}

/* ── Nota ── */
.contact__note {
  font-family: 'Jost', sans-serif;
  font-size: 12px;
  font-weight: 300;
  color: var(--ecora-sand);
  letter-spacing: 0.02em;
}

/* ── Éxito ── */
.contact__subscribed {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  padding: 2rem;
  border: 1px solid var(--ecora-sand);
  max-width: 480px;
  width: 100%;
}

.contact__subscribed-text {
  font-family: 'Cormorant Garamond', serif;
  font-size: 1.25rem;
  color: var(--ecora-black);
  font-style: italic;
}

.contact__subscribed-btn {
  font-family: 'Jost', sans-serif;
  font-size: 11px;
  font-weight: 500;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  color: var(--ecora-stone);
  background: none;
  border: none;
  cursor: pointer;
  text-decoration: underline;
  text-underline-offset: 3px;
}

/* ── Responsive ── */
@media (max-width: 600px) {
  .contact {
    padding: 5rem 1.25rem;
  }

  .contact__layout {
    flex-direction: column;
  }

  .contact__input {
    border-right: 1px solid var(--ecora-sand);
    border-bottom: none;
  }

  .contact__btn {
    text-align: center;
    padding: 0.875rem;
  }
}
</style>