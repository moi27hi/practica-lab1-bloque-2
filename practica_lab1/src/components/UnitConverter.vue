<script setup>
import { ref, computed, watch } from 'vue'

// ── Categorías y unidades ─────────────────────────────────────────────
const categories = {
  longitud: {
    label: '📏 Longitud',
    units: {
      km:       { label: 'Kilómetro (km)',    factor: 1000 },
      m:        { label: 'Metro (m)',          factor: 1 },
      cm:       { label: 'Centímetro (cm)',    factor: 0.01 },
      mm:       { label: 'Milímetro (mm)',     factor: 0.001 },
      pies:     { label: 'Pies (ft)',          factor: 0.3048 },
      pulgadas: { label: 'Pulgadas (in)',      factor: 0.0254 },
    }
  },
  peso: {
    label: '⚖️ Peso',
    units: {
      kg:  { label: 'Kilogramo (kg)', factor: 1 },
      g:   { label: 'Gramo (g)',      factor: 0.001 },
      mg:  { label: 'Miligramo (mg)', factor: 0.000001 },
      lb:  { label: 'Libra (lb)',     factor: 0.453592 },
      oz:  { label: 'Onza (oz)',      factor: 0.0283495 },
    }
  },
  temperatura: {
    label: '🌡️ Temperatura',
    units: {
      celsius:    { label: 'Celsius (°C)' },
      fahrenheit: { label: 'Fahrenheit (°F)' },
      kelvin:     { label: 'Kelvin (K)' },
    }
  }
}

// ── Estado ────────────────────────────────────────────────────────────
const activeCategory = ref('longitud')
const fromUnit       = ref('km')
const toUnit         = ref('m')
const inputValue     = ref('')
const result         = ref(null)

const currentUnits = computed(() => categories[activeCategory.value].units)

// Al cambiar categoría, resetear unidades y resultado
watch(activeCategory, () => {
  const keys = Object.keys(currentUnits.value)
  fromUnit.value = keys[0]
  toUnit.value   = keys[1]
  inputValue.value = ''
  result.value = null
})

// ── Conversión ────────────────────────────────────────────────────────
function convert() {
  const val = parseFloat(inputValue.value)
  if (isNaN(val)) { result.value = null; return }

  if (activeCategory.value === 'temperatura') {
    result.value = convertTemp(val, fromUnit.value, toUnit.value)
  } else {
    // Convertir a unidad base (metros o kg) y luego a destino
    const base = val * currentUnits.value[fromUnit.value].factor
    result.value = base / currentUnits.value[toUnit.value].factor
  }
}

function convertTemp(val, from, to) {
  // Primero a Celsius
  let celsius
  if (from === 'celsius')    celsius = val
  if (from === 'fahrenheit') celsius = (val - 32) * 5 / 9
  if (from === 'kelvin')     celsius = val - 273.15

  // Luego a destino
  if (to === 'celsius')    return celsius
  if (to === 'fahrenheit') return celsius * 9 / 5 + 32
  if (to === 'kelvin')     return celsius + 273.15
}

function swap() {
  const tmp    = fromUnit.value
  fromUnit.value = toUnit.value
  toUnit.value   = tmp
  if (result.value !== null) {
    inputValue.value = formatResult(result.value)
    convert()
  }
}

function formatResult(n) {
  if (n === null) return ''

  if (n === 0) return '0'

  return parseFloat(n.toFixed(2)).toString()
}

const displayResult = computed(() =>
  result.value !== null ? formatResult(result.value) : '—'
)

const resultUnit = computed(() =>
  currentUnits.value[toUnit.value]?.label.split(' ')[0] ?? ''
)
</script>

<template>
  <div class="converter">
    <h2 class="title">Conversor</h2>

    <!-- Tabs de categoría -->
    <div class="tabs">
      <button
        v-for="(cat, key) in categories"
        :key="key"
        :class="{ active: activeCategory === key }"
        @click="activeCategory = key"
      >
        {{ cat.label }}
      </button>
    </div>

    <!-- Formulario -->
    <div class="form">
      <!-- Valor de entrada -->
      <input
        v-model="inputValue"
        type="number"
        placeholder="Ingresa un valor"
        @input="convert"
      />

      <!-- Selects + swap -->
      <div class="selects">
        <select v-model="fromUnit" @change="convert">
          <option v-for="(unit, key) in currentUnits" :key="key" :value="key">
            {{ unit.label }}
          </option>
        </select>

        <button class="swap-btn" @click="swap" title="Intercambiar">
          ⇄
        </button>

        <select v-model="toUnit" @change="convert">
          <option v-for="(unit, key) in currentUnits" :key="key" :value="key">
            {{ unit.label }}
          </option>
        </select>
      </div>
    </div>

    <!-- Resultado -->
    <div class="result-box">
      <span class="result-value">{{ displayResult }}</span>
      <span class="result-unit">{{ result !== null ? resultUnit : '' }}</span>
    </div>

  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Sans:wght@300;400;500&display=swap');

.converter {
  width: 320px;
  border-radius: 28px;
  background: #18181d;
  box-shadow: 0 30px 80px rgba(0,0,0,.7), 0 0 0 1px #2a2a33;
  padding: 1.6rem 1.4rem;
  font-family: 'DM Sans', sans-serif;
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
}

/* ── Título ─── */
.title {
  font-family: 'Syne', sans-serif;
  font-size: 1.6rem;
  font-weight: 800;
  color: #f0f0f0;
  letter-spacing: -0.5px;
  text-align: center;
}

/* ── Tabs ─── */
.tabs {
  display: flex;
  gap: .4rem;
  flex-wrap: wrap;
  justify-content: center;
}
.tabs button {
  background: #2a2a33;
  border: 1.5px solid #2a2a33;
  color: #7a7a8a;
  border-radius: 10px;
  padding: .35rem .7rem;
  font-family: 'DM Sans', sans-serif;
  font-size: .78rem;
  cursor: pointer;
  transition: all .15s;
  white-space: nowrap;
}
.tabs button:hover  { border-color: #3a3a46; color: #f0f0f0; }
.tabs button.active { background: #c8f058; border-color: #c8f058; color: #111; font-weight: 500; }

/* ── Form ─── */
.form { display: flex; flex-direction: column; gap: .7rem; }

input[type="number"] {
  width: 100%;
  background: #111116;
  border: 1.5px solid #2a2a33;
  border-radius: 14px;
  padding: .8rem 1rem;
  color: #f0f0f0;
  font-family: 'Syne', sans-serif;
  font-size: 1.4rem;
  font-weight: 700;
  outline: none;
  transition: border-color .2s;
  -moz-appearance: textfield;
}
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button { -webkit-appearance: none; }
input:focus { border-color: #c8f058; }
input::placeholder { color: #3a3a46; font-size: 1rem; font-weight: 300; }

/* ── Selects row ─── */
.selects {
  display: flex;
  align-items: center;
  gap: .5rem;
}

select {
  flex: 1;
  background: #1f1f26;
  border: 1.5px solid #2a2a33;
  border-radius: 12px;
  padding: .55rem .7rem;
  color: #f0f0f0;
  font-family: 'DM Sans', sans-serif;
  font-size: .82rem;
  outline: none;
  cursor: pointer;
  transition: border-color .2s;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 24 24' fill='none' stroke='%237a7a8a' stroke-width='2.5'%3E%3Cpolyline points='6 9 12 15 18 9'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right .6rem center;
  padding-right: 1.8rem;
}
select:focus { border-color: #c8f058; }
select option { background: #1f1f26; }

.swap-btn {
  flex-shrink: 0;
  background: #2a2a33;
  border: 1.5px solid #2a2a33;
  color: #c8f058;
  border-radius: 10px;
  width: 36px;
  height: 36px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background .15s, transform .2s;
  display: flex; align-items: center; justify-content: center;
  padding: 0;
}
.swap-btn:hover { background: #333340; transform: rotate(180deg); }

/* ── Resultado ─── */
.result-box {
  background: #111116;
  border: 1.5px solid #2a2a33;
  border-radius: 14px;
  padding: 1rem 1.2rem;
  display: flex;
  align-items: baseline;
  justify-content: flex-end;
  gap: .5rem;
  min-height: 64px;
}
.result-value {
  font-family: 'Syne', sans-serif;
  font-size: 1.9rem;
  font-weight: 800;
  color: #c8f058;
  letter-spacing: -1px;
  word-break: break-all;
  text-align: right;
}
.result-unit {
  font-size: .82rem;
  color: #7a7a8a;
  white-space: nowrap;
}
</style>