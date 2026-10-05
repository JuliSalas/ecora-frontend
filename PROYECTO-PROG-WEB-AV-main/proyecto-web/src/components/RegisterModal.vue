<script setup>
import { reactive, ref } from 'vue'

const emit = defineEmits(['close', 'registered'])

const formulario = reactive({
  nombre: '',
  correo: '',
  contrasenna: '',
  confirmarContrasenna: ''
})

const errores = reactive({
  nombre: '',
  correo: '',
  contrasenna: '',
  confirmarContrasenna: ''
})

const mensajeExito = ref('')
const mostrandoContrasenna = ref(false)

const limpiarErrores = () => {
  errores.nombre = ''
  errores.correo = ''
  errores.contrasenna = ''
  errores.confirmarContrasenna = ''
  mensajeExito.value = ''
}

const validarCorreo = (correo) => {
  const expresionCorreo = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return expresionCorreo.test(correo)
}

const validarFormulario = () => {
  limpiarErrores()

  let formularioValido = true

  if (!formulario.nombre.trim()) {
    errores.nombre = 'El nombre completo es obligatorio.'
    formularioValido = false
  } else if (formulario.nombre.trim().length < 3) {
    errores.nombre = 'El nombre debe tener al menos 3 caracteres.'
    formularioValido = false
  }

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
  } else if (formulario.contrasenna.length < 8) {
    errores.contrasenna = 'La contraseña debe tener al menos 8 caracteres.'
    formularioValido = false
  } else if (!/[A-Z]/.test(formulario.contrasenna)) {
    errores.contrasenna =
      'La contraseña debe contener al menos una letra mayúscula.'
    formularioValido = false
  } else if (!/[a-z]/.test(formulario.contrasenna)) {
    errores.contrasenna =
      'La contraseña debe contener al menos una letra minúscula.'
    formularioValido = false
  } else if (!/[0-9]/.test(formulario.contrasenna)) {
    errores.contrasenna =
      'La contraseña debe contener al menos un número.'
    formularioValido = false
  }

  if (!formulario.confirmarContrasenna) {
    errores.confirmarContrasenna = 'Debe confirmar la contraseña.'
    formularioValido = false
  } else if (
    formulario.confirmarContrasenna !== formulario.contrasenna
  ) {
    errores.confirmarContrasenna = 'Las contraseñas no coinciden.'
    formularioValido = false
  }

  return formularioValido
}

const registrarUsuario = async () => {
  if (!validarFormulario()) {
    return
  }

  const correoNormalizado = formulario.correo.trim().toLowerCase()

  try {
    const response = await fetch('/api/usuarios', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        nombre: formulario.nombre.trim(),
        correo: correoNormalizado,
        contrasenna: formulario.contrasenna,
        tipoUsuario: 'cliente'
      })
    })

    const resultado = await response.json()

    if (!response.ok) {
      if (response.status === 409) {
        errores.correo =
          'Ya existe una cuenta registrada con este correo.'
        return
      }

      throw new Error(
        resultado.msj || 'No fue posible registrar el usuario.'
      )
    }

    mensajeExito.value = 'La cuenta fue creada correctamente.'

    emit('registered', resultado.usuario || resultado)

    formulario.nombre = ''
    formulario.correo = ''
    formulario.contrasenna = ''
    formulario.confirmarContrasenna = ''
  } catch (error) {
    console.error('Error al registrar usuario:', error)

    errores.correo =
      error.message || 'No fue posible registrar la cuenta.'
  }
}

const cerrarModal = () => {
  emit('close')
}
</script>

<template>
  <div class="register-overlay" @click.self="cerrarModal">
    <div class="register-modal">
      <button
        class="close-button"
        type="button"
        aria-label="Cerrar formulario"
        @click="cerrarModal"
      >
        ×
      </button>

      <div class="register-header">
        <p class="register-label">ÚNETE A ECORA</p>
        <h2>Crear una cuenta</h2>
        <p>
          Regístrate para realizar compras y consultar tus pedidos.
        </p>
      </div>

      <form novalidate @submit.prevent="registrarUsuario">
        <div class="mb-3">
          <label for="nombre" class="form-label">
            Nombre completo
          </label>

          <input
            id="nombre"
            v-model="formulario.nombre"
            type="text"
            class="form-control"
            :class="{ 'is-invalid': errores.nombre }"
            placeholder="Digite su nombre completo"
          />

          <div class="invalid-feedback">
            {{ errores.nombre }}
          </div>
        </div>

        <div class="mb-3">
          <label for="correo" class="form-label">
            Correo electrónico
          </label>

          <input
            id="correo"
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
          <label for="contrasenna" class="form-label">
            Contraseña
          </label>

          <div class="password-container">
            <input
              id="contrasenna"
              v-model="formulario.contrasenna"
              :type="mostrandoContrasenna ? 'text' : 'password'"
              class="form-control"
              :class="{ 'is-invalid': errores.contrasenna }"
              placeholder="Mínimo 8 caracteres"
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

        <div class="mb-3">
          <label for="confirmarContrasenna" class="form-label">
            Confirmar contraseña
          </label>

          <input
            id="confirmarContrasenna"
            v-model="formulario.confirmarContrasenna"
            :type="mostrandoContrasenna ? 'text' : 'password'"
            class="form-control"
            :class="{
              'is-invalid': errores.confirmarContrasenna
            }"
            placeholder="Digite nuevamente la contraseña"
          />

          <div class="invalid-feedback">
            {{ errores.confirmarContrasenna }}
          </div>
        </div>

        <p v-if="mensajeExito" class="alert alert-success">
          {{ mensajeExito }}
        </p>

        <button class="register-button" type="submit">
          Crear cuenta
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
.register-overlay {
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

.register-modal {
  position: relative;
  width: 100%;
  max-width: 520px;
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

.register-header {
  margin-bottom: 28px;
  text-align: center;
}

.register-label {
  margin-bottom: 8px;
  color: var(--ecora-warm);
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 3px;
}

.register-header h2 {
  margin-bottom: 10px;
  font-size: 32px;
}

.register-header p {
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

.register-button {
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

.register-button:hover {
  background: transparent;
  color: var(--ecora-black);
}

@media (max-width: 576px) {
  .register-overlay {
    padding: 12px;
    align-items: flex-start;
  }

  .register-modal {
    margin-top: 20px;
    padding: 38px 22px 28px;
  }

  .register-header h2 {
    font-size: 27px;
  }
}
</style>