<script setup>
import { ref, computed } from 'vue'

// ── Estado ───────────────────────────────────────────────────────────
const display     = ref('0')
const firstValue  = ref(null)
const operator    = ref(null)
const waitingNext = ref(false)
const expression  = ref('')

// ── Helpers ──────────────────────────────────────────────────────────
const isOverflow = computed(() => display.value.replace('-', '').length > 9)

function format(num) {
  const n = parseFloat(num)
  if (isNaN(n)) return 'Error'
  return parseFloat(n.toPrecision(10)).toString()
}

// ── Acciones ─────────────────────────────────────────────────────────
function pressDigit(digit) {
  if (waitingNext.value) {
    display.value = digit
    waitingNext.value = false
  } else {
    if (display.value === '0' && digit !== '.') {
      display.value = digit
    } else if (digit === '.' && display.value.includes('.')) {
      return
    } else if (display.value.replace('-', '').length >= 10) {
      return
    } else {
      display.value += digit
    }
  }
}

function pressOperator(op) {
  const current = parseFloat(display.value)
  if (firstValue.value !== null && !waitingNext.value) {
    const result = calculate(firstValue.value, current, operator.value)
    display.value = format(result)
    firstValue.value = parseFloat(display.value)
  } else {
    firstValue.value = current
  }
  operator.value    = op
  waitingNext.value = true
  expression.value  = `${format(firstValue.value)} ${op}`
}

function calculate(a, b, op) {
  switch (op) {
    case '+': return a + b
    case '−': return a - b
    case '×': return a * b
    case '÷': return b === 0 ? 'Error' : a / b
    default:  return b
  }
}

function pressEquals() {
  if (operator.value === null || firstValue.value === null) return
  const current = parseFloat(display.value)
  expression.value  = `${format(firstValue.value)} ${operator.value} ${format(current)} =`
  const result      = calculate(firstValue.value, current, operator.value)
  display.value     = result === 'Error' ? 'Error' : format(result)
  firstValue.value  = null
  operator.value    = null
  waitingNext.value = true
}

function pressClear() {
  display.value     = '0'
  firstValue.value  = null
  operator.value    = null
  waitingNext.value = false
  expression.value  = ''
}

function pressToggleSign() {
  if (display.value === '0' || display.value === 'Error') return
  display.value = display.value.startsWith('-')
    ? display.value.slice(1)
    : '-' + display.value
}

function pressPercent() {
  const n = parseFloat(display.value)
  if (isNaN(n)) return
  display.value = format(n / 100)
}

function handleKey(e) {
  if (e.key >= '0' && e.key <= '9') pressDigit(e.key)
  else if (e.key === '.')           pressDigit('.')
  else if (e.key === '+')           pressOperator('+')
  else if (e.key === '-')           pressOperator('−')
  else if (e.key === '*')           pressOperator('×')
  else if (e.key === '/')           { e.preventDefault(); pressOperator('÷') }
  else if (e.key === 'Enter' || e.key === '=') pressEquals()
  else if (e.key === 'Escape')      pressClear()
  else if (e.key === 'Backspace') {
    if (display.value.length > 1 && display.value !== 'Error') {
      display.value = display.value.slice(0, -1) || '0'
    } else {
      display.value = '0'
    }
  }
}
</script>

<template>
  <div class="calc" @keydown="handleKey" tabindex="0">

    <!-- Pantalla -->
    <div class="screen">
      <span class="expression">{{ expression }}</span>
      <span class="value" :class="{ small: isOverflow }">{{ display }}</span>
    </div>

    <!-- Botones -->
    <div class="grid">
      <button class="fn"  @click="pressClear">{{ firstValue !== null || display !== '0' ? 'C' : 'AC' }}</button>
      <button class="fn"  @click="pressToggleSign">+/−</button>
      <button class="fn"  @click="pressPercent">%</button>
      <button class="op" :class="{ active: operator === '÷' }" @click="pressOperator('÷')">÷</button>

      <button @click="pressDigit('7')">7</button>
      <button @click="pressDigit('8')">8</button>
      <button @click="pressDigit('9')">9</button>
      <button class="op" :class="{ active: operator === '×' }" @click="pressOperator('×')">×</button>

      <button @click="pressDigit('4')">4</button>
      <button @click="pressDigit('5')">5</button>
      <button @click="pressDigit('6')">6</button>
      <button class="op" :class="{ active: operator === '−' }" @click="pressOperator('−')">−</button>

      <button @click="pressDigit('1')">1</button>
      <button @click="pressDigit('2')">2</button>
      <button @click="pressDigit('3')">3</button>
      <button class="op" :class="{ active: operator === '+' }" @click="pressOperator('+')">+</button>

      <button class="zero" @click="pressDigit('0')">0</button>
      <button @click="pressDigit('.')">.</button>
      <button class="eq" @click="pressEquals">=</button>
    </div>

  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Sans:wght@300;400&display=swap');

.calc {
  width: 300px;
  min-width: 300px;
  max-width: 300px;
  border-radius: 28px;
  background: #18181d;
  box-shadow: 0 30px 80px rgba(0,0,0,.7), 0 0 0 1px #2a2a33;
  overflow: hidden;
  user-select: none;
  outline: none;
  font-family: 'DM Sans', sans-serif;
}

/* ── Pantalla ─── */
.screen {
  padding: 1.4rem 1.5rem 1rem;
  background: #111116;
  min-height: 110px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-end;
  gap: .2rem;
  border-bottom: 1px solid #2a2a33;
}
.expression {
  font-size: .78rem;
  color: #5a5a6a;
  min-height: 1.2em;
  letter-spacing: .3px;
}
.value {
  font-family: 'Syne', sans-serif;
  font-size: 2.8rem;
  font-weight: 800;
  color: #f0f0f0;
  letter-spacing: -1.5px;
  line-height: 1;
  transition: font-size .15s;
  word-break: break-all;
  text-align: right;
}
.value.small { font-size: 1.9rem; letter-spacing: -.5px; }

/* ── Grid ─── */
.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1px;
  background: #2a2a33;
}

/* ── Botones ─── */
button {
  background: #1f1f26;
  border: none;
  color: #f0f0f0;
  font-family: 'DM Sans', sans-serif;
  font-size: 1.25rem;
  font-weight: 400;
  padding: 0;
  height: 68px;
  cursor: pointer;
  transition: background .12s, color .12s, transform .08s;
  -webkit-tap-highlight-color: transparent;
}
button:hover  { background: #28282f; }
button:active { transform: scale(.93); }

.fn { background: #2e2e38; color: #c8f058; font-weight: 500; }
.fn:hover { background: #373742; }

.op { background: #2a2a33; color: #c8f058; font-size: 1.5rem; font-weight: 700; }
.op:hover  { background: #333340; }
.op.active { background: #c8f058; color: #111; }

.zero { grid-column: span 2; text-align: left; padding-left: 1.6rem; }

.eq { background: #c8f058; color: #111; font-size: 1.5rem; font-weight: 700; }
.eq:hover  { background: #d4f570; }
.eq:active { background: #b8e040; }
</style>