<script setup>
import Calculator    from './components/Calculator.vue' 
import UnitConverter from './components/UnitConverter.vue'
import ClientForm    from './components/ClientForm.vue'
</script>


<template>
  <div class="wrapper">
    <div class="row">
      <Calculator />
      <UnitConverter />
    </div>
    <div class="row">
      <ClientForm />
    </div>
  </div>
</template>

<style scoped>
.wrapper {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2rem;
  padding: 3rem 1rem;
  background: #0b0b0e;
}
.row {
  display: flex;
  gap: 2rem;
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
}
</style>