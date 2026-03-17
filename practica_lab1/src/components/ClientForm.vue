<script setup>
import { ref, reactive, computed } from 'vue'

// ── Estado del formulario ─────────────────────────────────────────────
const form = reactive({
  nombre:     '',
  apellido:   '',
  email:      '',
  telefono:   '',
  direccion:  '',
  nacimiento: '',
})

const submitted  = ref(false)
const showSuccess = ref(false)
const clientes   = ref([])

// ── Validaciones ──────────────────────────────────────────────────────
const errors = computed(() => {
  const e = {}
  if (submitted.value) {
    if (!form.nombre.trim())     e.nombre     = 'El nombre es requerido'
    if (!form.apellido.trim())   e.apellido   = 'El apellido es requerido'
    if (!form.email.trim())      e.email      = 'El correo es requerido'
    else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email))
                                  e.email      = 'Correo no válido'
    if (!form.telefono.trim())   e.telefono   = 'El teléfono es requerido'
    else if (!/^\+?[\d\s\-]{7,15}$/.test(form.telefono))
                                  e.telefono   = 'Teléfono no válido'
    if (!form.direccion.trim())  e.direccion  = 'La dirección es requerida'
    if (!form.nacimiento)        e.nacimiento = 'La fecha es requerida'
  }
  return e
})

const isValid = computed(() => Object.keys(errors.value).length === 0)

// ── Envío ─────────────────────────────────────────────────────────────
function handleSubmit() {
  submitted.value = true
  if (!isValid.value) return

  // Agregar a lista
  clientes.value.unshift({
    id:         Date.now(),
    nombre:     form.nombre,
    apellido:   form.apellido,
    email:      form.email,
    telefono:   form.telefono,
    direccion:  form.direccion,
    nacimiento: form.nacimiento,
  })

  // Mostrar éxito
  showSuccess.value = true
  setTimeout(() => showSuccess.value = false, 3000)

  // Limpiar formulario
  Object.assign(form, {
    nombre: '', apellido: '', email: '',
    telefono: '', direccion: '', nacimiento: '',
  })
  submitted.value = false
}

function removeCliente(id) {
  clientes.value = clientes.value.filter(c => c.id !== id)
}

function formatDate(d) {
  if (!d) return ''
  const [y, m, day] = d.split('-')
  return `${day}/${m}/${y}`
}
</script>

<template>
  <div class="wrapper">

    <!-- Formulario -->
    <div class="card form-card">
      <h2 class="card-title">Nuevo cliente</h2>
      <p class="card-sub">Completa todos los campos para registrar</p>

      <!-- Toast éxito -->
      <Transition name="toast">
        <div v-if="showSuccess" class="toast">
          ✦ Cliente registrado exitosamente
        </div>
      </Transition>

      <form @submit.prevent="handleSubmit" novalidate>

        <!-- Nombre + Apellido -->
        <div class="row-2">
          <div class="field">
            <label>Nombre</label>
            <input v-model="form.nombre" type="text" placeholder="Ej: Juan" :class="{ error: errors.nombre }" />
            <span class="err-msg" v-if="errors.nombre">{{ errors.nombre }}</span>
          </div>
          <div class="field">
            <label>Apellido</label>
            <input v-model="form.apellido" type="text" placeholder="Ej: Pérez" :class="{ error: errors.apellido }" />
            <span class="err-msg" v-if="errors.apellido">{{ errors.apellido }}</span>
          </div>
        </div>

        <!-- Email -->
        <div class="field">
          <label>Correo electrónico</label>
          <input v-model="form.email" type="email" placeholder="correo@ejemplo.com" :class="{ error: errors.email }" />
          <span class="err-msg" v-if="errors.email">{{ errors.email }}</span>
        </div>

        <!-- Teléfono -->
        <div class="field">
          <label>Teléfono</label>
          <input v-model="form.telefono" type="tel" placeholder="+503 7000-0000" :class="{ error: errors.telefono }" />
          <span class="err-msg" v-if="errors.telefono">{{ errors.telefono }}</span>
        </div>

        <!-- Dirección -->
        <div class="field">
          <label>Dirección</label>
          <input v-model="form.direccion" type="text" placeholder="Calle, ciudad, país" :class="{ error: errors.direccion }" />
          <span class="err-msg" v-if="errors.direccion">{{ errors.direccion }}</span>
        </div>

        <!-- Fecha de nacimiento -->
        <div class="field">
          <label>Fecha de nacimiento</label>
          <input v-model="form.nacimiento" type="date" :class="{ error: errors.nacimiento }" />
          <span class="err-msg" v-if="errors.nacimiento">{{ errors.nacimiento }}</span>
        </div>

        <button type="submit" class="btn-submit">
          Registrar cliente →
        </button>

      </form>
    </div>

    <!-- Lista de clientes -->
    <div class="card list-card" v-if="clientes.length > 0">
      <h2 class="card-title">Clientes registrados
        <span class="badge">{{ clientes.length }}</span>
      </h2>

      <TransitionGroup name="list" tag="ul" class="client-list">
        <li v-for="c in clientes" :key="c.id" class="client-item">
          <div class="client-avatar">{{ c.nombre[0] }}{{ c.apellido[0] }}</div>
          <div class="client-info">
            <span class="client-name">{{ c.nombre }} {{ c.apellido }}</span>
            <span class="client-detail">{{ c.email }}</span>
            <span class="client-detail">{{ c.telefono }} · {{ formatDate(c.nacimiento) }}</span>
            <span class="client-detail muted">{{ c.direccion }}</span>
          </div>
          <button class="remove-btn" @click="removeCliente(c.id)" title="Eliminar">✕</button>
        </li>
      </TransitionGroup>
    </div>

  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Sans:wght@300;400;500&display=swap');

/* ── Layout ─── */
.wrapper {
  display: flex;
  gap: 1.5rem;
  align-items: flex-start;
  flex-wrap: wrap;
  font-family: 'DM Sans', sans-serif;
}

/* ── Card base ─── */
.card {
  background: #18181d;
  border: 1.5px solid #2a2a33;
  border-radius: 24px;
  padding: 1.8rem;
  box-shadow: 0 20px 60px rgba(0,0,0,.5);
}
.form-card { width: 380px; }
.list-card { width: 340px; }

/* ── Títulos ─── */
.card-title {
  font-family: 'Syne', sans-serif;
  font-size: 1.4rem;
  font-weight: 800;
  color: #f0f0f0;
  letter-spacing: -.5px;
  display: flex;
  align-items: center;
  gap: .5rem;
  margin-bottom: .25rem;
}
.card-sub {
  font-size: .82rem;
  color: #5a5a6a;
  margin-bottom: 1.4rem;
}

/* ── Badge ─── */
.badge {
  background: #c8f058;
  color: #111;
  font-size: .72rem;
  font-weight: 700;
  border-radius: 20px;
  padding: .1rem .5rem;
}

/* ── Toast ─── */
.toast {
  background: #c8f058;
  color: #111;
  border-radius: 12px;
  padding: .7rem 1rem;
  font-size: .85rem;
  font-weight: 500;
  margin-bottom: 1.2rem;
  text-align: center;
}
.toast-enter-active, .toast-leave-active { transition: all .3s ease; }
.toast-enter-from, .toast-leave-to { opacity: 0; transform: translateY(-8px); }

/* ── Form fields ─── */
form { display: flex; flex-direction: column; gap: .9rem; }

.row-2 { display: grid; grid-template-columns: 1fr 1fr; gap: .7rem; }

.field { display: flex; flex-direction: column; gap: .3rem; }

label {
  font-size: .78rem;
  font-weight: 500;
  color: #7a7a8a;
  letter-spacing: .3px;
  text-transform: uppercase;
}

input {
  background: #111116;
  border: 1.5px solid #2a2a33;
  border-radius: 12px;
  padding: .7rem .9rem;
  color: #f0f0f0;
  font-family: 'DM Sans', sans-serif;
  font-size: .92rem;
  outline: none;
  transition: border-color .2s;
  width: 100%;
}
input::placeholder { color: #3a3a46; }
input:focus  { border-color: #c8f058; }
input.error  { border-color: #ff5e6c; }
input[type="date"]::-webkit-calendar-picker-indicator { filter: invert(.4); cursor: pointer; }

.err-msg {
  font-size: .73rem;
  color: #ff5e6c;
  padding-left: .2rem;
}

/* ── Submit ─── */
.btn-submit {
  margin-top: .4rem;
  background: #c8f058;
  border: none;
  border-radius: 14px;
  padding: .85rem;
  color: #111;
  font-family: 'Syne', sans-serif;
  font-size: .95rem;
  font-weight: 700;
  cursor: pointer;
  transition: opacity .2s, transform .15s;
  letter-spacing: -.2px;
}
.btn-submit:hover  { opacity: .85; transform: translateY(-1px); }
.btn-submit:active { transform: translateY(0); }

/* ── Lista clientes ─── */
.client-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: .6rem;
  margin-top: 1rem;
}

.client-item {
  display: flex;
  align-items: flex-start;
  gap: .8rem;
  background: #111116;
  border: 1.5px solid #2a2a33;
  border-radius: 14px;
  padding: .8rem;
  animation: slide-in .22s ease;
}

@keyframes slide-in {
  from { opacity: 0; transform: translateY(-6px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* Avatar */
.client-avatar {
  flex-shrink: 0;
  width: 38px; height: 38px;
  border-radius: 10px;
  background: #c8f058;
  color: #111;
  font-family: 'Syne', sans-serif;
  font-size: .85rem;
  font-weight: 800;
  display: flex; align-items: center; justify-content: center;
  text-transform: uppercase;
}

.client-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: .15rem;
  min-width: 0;
}
.client-name {
  font-size: .9rem;
  font-weight: 500;
  color: #f0f0f0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.client-detail {
  font-size: .76rem;
  color: #7a7a8a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.client-detail.muted { color: #4a4a56; }

.remove-btn {
  flex-shrink: 0;
  background: transparent;
  border: none;
  color: #3a3a46;
  font-size: .8rem;
  cursor: pointer;
  padding: .2rem .3rem;
  border-radius: 6px;
  transition: color .15s;
  line-height: 1;
}
.remove-btn:hover { color: #ff5e6c; }

/* TransitionGroup */
.list-enter-active, .list-leave-active { transition: all .25s ease; }
.list-enter-from { opacity: 0; transform: translateY(-8px); }
.list-leave-to   { opacity: 0; transform: translateX(20px); }
.list-move       { transition: transform .25s ease; }
</style>