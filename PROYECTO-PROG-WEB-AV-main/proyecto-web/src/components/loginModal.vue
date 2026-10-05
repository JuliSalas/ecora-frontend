<script setup>
import { reactive, ref } from 'vue'

const emit = defineEmits(['close', 'login-success'])

const formulario = reactive({
  correo: '',
  contrasenna: ''
})

const errores = reactive({
  correo: '',
  contrasenna: '',
  general: ''
})

const mostrandoContrasenna = ref(false)

const limpiarErrores = () => {
  errores.correo = ''
  errores.contrasenna = ''
  errores.general = ''
}

const validarCorreo = (correo) => {
  const expresionCorreo = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return expresionCorreo.test(correo)
}

const validarFormulario = () => {
  limpiarErrores()

  let formularioValido = true

  if (!formulario.correo.trim()) {
    errores.correo = 'El correo electrónico es obligatorio.'
    formularioValido = false
  } else if (!validarCorreo(formulario.correo)) {
    errores.correo = 'Ingrese un correo electrónico válido.'
    formularioValido = false
  }

  if (!formulario.contrasenna) {
    errores.contrasenna = 'La contraseña es obligatoria.'
    formularioValido = false
  }

  return formularioValido
}

const iniciarSesion = async () => {
  if (!validarFormulario()) {
    return
  }

  limpiarErrores()

  const correoNormalizado = formulario.correo.trim().toLowerCase()

  try {
    const response = await fetch('/api/usuarios/iniciar-sesion', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        correo: correoNormalizado,
        contrasenna: formulario.contrasenna
      })
    })

    const resultado = await response.json()

    console.log('STATUS:', response.status)
    console.log('RESPUESTA LOGIN:', resultado)

    if (!response.ok) {
      errores.general =
        resultado.msj ||
        resultado.msg ||
        'El correo o la contraseña son incorrectos.'
      return
    }

    const sesionUsuario = {
      id: resultado.usuario.id,
      nombre: resultado.usuario.nombre,
      correo: resultado.usuario.correo,
      rol: resultado.usuario.rol
    }

    localStorage.setItem(
      'ecoraSesion',
      JSON.stringify(sesionUsuario)
    )

    localStorage.setItem(
      'ecoraToken',
      resultado.token
    )

    emit('login-success', sesionUsuario)

  } catch (error) {
    console.error('Error al iniciar sesión:', error)

    errores.general =
      'No fue posible conectar con el servidor.'
  }
}

const cerrarModal = () => {
  emit('close')
}
</script>

<template>
  <div class="login-overlay" @click.self="cerrarModal">
    <div class="login-modal">
      <button
        class="close-button"
        type="button"
        aria-label="Cerrar formulario"
        @click="cerrarModal"
      >
        ×
      </button>

      <div class="login-header">
        <p class="login-label">BIENVENIDO DE NUEVO</p>

        <h2>Iniciar sesión</h2>

        <p>
          Ingresa tus datos para acceder a tu cuenta.
        </p>
      </div>

      <form novalidate @submit.prevent="iniciarSesion">
        <div class="mb-3">
          <label for="loginCorreo" class="form-label">
            Correo electrónico
          </label>

          <input
            id="loginCorreo"
            v-model="formulario.correo"
            type="email"
            class="form-control"
            :class="{ 'is-invalid': errores.correo }"
            placeholder="nombre@correo.com"
          />

          <div class="invalid-feedback">
            {{ errores.correo }}
          </div>
        </div>

        <div class="mb-3">
          <label for="loginContrasenna" class="form-label">
            Contraseña
          </label>

          <div class="password-container">
            <input
              id="loginContrasenna"
              v-model="formulario.contrasenna"
              :type="mostrandoContrasenna ? 'text' : 'password'"
              class="form-control"
              :class="{ 'is-invalid': errores.contrasenna }"
              placeholder="Ingrese su contraseña"
            />

            <button
              class="password-button"
              type="button"
              @click="mostrandoContrasenna = !mostrandoContrasenna"
            >
              {{ mostrandoContrasenna ? 'Ocultar' : 'Mostrar' }}
            </button>

            <div class="invalid-feedback">
              {{ errores.contrasenna }}
            </div>
          </div>
        </div>

        <div
          v-if="errores.general"
          class="alert alert-danger"
          role="alert"
        >
          {{ errores.general }}
        </div>

        <button class="login-button" type="submit">
          Iniciar sesión
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.login-overlay {
  position: fixed;
  inset: 0;
  z-index: 2000;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px;
  background: rgba(11, 11, 9, 0.72);
  overflow-y: auto;
}

.login-modal {
  position: relative;
  width: 100%;
  max-width: 500px;
  padding: 44px;
  border-radius: 4px;
  background: var(--ecora-cream);
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.close-button {
  position: absolute;
  top: 12px;
  right: 18px;
  border: none;
  background: transparent;
  color: var(--ecora-black);
  font-size: 32px;
  line-height: 1;
  cursor: pointer;
}

.login-header {
  margin-bottom: 28px;
  text-align: center;
}

.login-label {
  margin-bottom: 8px;
  color: var(--ecora-warm);
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 3px;
}

.login-header h2 {
  margin-bottom: 10px;
  font-size: 32px;
}

.login-header p {
  color: var(--ecora-stone);
}

.form-label {
  font-weight: 500;
}

.form-control {
  min-height: 46px;
  border: 1px solid var(--ecora-sand);
  border-radius: 2px;
  background: #fff;
}

.form-control:focus {
  border-color: var(--ecora-warm);
  box-shadow: 0 0 0 0.2rem rgba(111, 95, 72, 0.15);
}

.password-container {
  position: relative;
}

.password-container .form-control {
  padding-right: 90px;
}

.password-button {
  position: absolute;
  top: 11px;
  right: 12px;
  border: none;
  background: transparent;
  color: var(--ecora-warm);
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
}

.password-container .form-control.is-invalid {
  background-position: right 90px center;
}

.login-button {
  width: 100%;
  margin-top: 10px;
  padding: 13px 24px;
  border: 1px solid var(--ecora-black);
  border-radius: 2px;
  background: var(--ecora-black);
  color: var(--ecora-cream);
  font-weight: 600;
  letter-spacing: 1px;
  transition:
    background 0.2s ease,
    color 0.2s ease;
}

.login-button:hover {
  background: transparent;
  color: var(--ecora-black);
}

@media (max-width: 576px) {
  .login-overlay {
    padding: 12px;
    align-items: flex-start;
  }

  .login-modal {
    margin-top: 20px;
    padding: 38px 22px 28px;
  }

  .login-header h2 {
    font-size: 27px;
  }
}
</style>