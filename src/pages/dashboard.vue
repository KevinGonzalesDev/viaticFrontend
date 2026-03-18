<script setup>
import AnalyticsBudget from '@/views/dashboard/AnalyticsBudget.vue'
import { onMounted, ref } from 'vue'


const saludo = ref('')

onMounted(async () => {
  document.title = 'Dashboard - Viaticos'
  const storedUser = localStorage.getItem('user')
  if (storedUser) {
    const user = JSON.parse(storedUser)
    saludo.value = `Información de, ${user.name}`
  }
})

</script>

<template>
  <VRow class="match-height">
    <VCol cols="12">
      <h2 class="text-h4 font-weight-bold mb-0">
        {{ saludo }}
      </h2>
    </VCol>
    <VCol cols="12">
      <p class="mb-0">
        Bienvenido al panel de control de Viaticos, aquí puedes encontrar un resumen de tus actividades recientes,
        estadísticas y acceso rápido a las funciones principales.
      </p>
    </VCol>
    <VCol cols="12" md="4">
      <AnalyticsBudget />
    </VCol>
    <!-- <VCol cols="12" md="8">
      <AnalyticsDepositWithdraw />
    </VCol> -->



  </VRow>
</template>
