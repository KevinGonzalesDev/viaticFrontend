<script setup>
import { onMounted, ref } from 'vue'

const totalProfit = {
  title: 'Total Profit',
  color: 'secondary',
  icon: 'ri-pie-chart-2-line',
  stats: '$25.6k',
  change: 42,
  subtitle: 'Weekly Project',
}

const newProject = {
  title: 'New Project',
  color: 'primary',
  icon: 'ri-file-word-2-line',
  stats: '862',
  change: -18,
  subtitle: 'Yearly Project',
}

const saludo = ref('')

onMounted(() => {
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
    <VCol cols="12">
      <VCard>
        <VCardText>
          <p class="mb-0">
            Defina sus parametros a rastrear.
          </p>
        </VCardText>
      </VCard>
    </VCol>

    <!-- <VCol cols="12">
          <UpgradeToPro />
        </VCol> -->

    <!-- <VCol cols="12" md="4">
      <AnalyticsAward />
    </VCol>

    <VCol cols="12" md="8">
      <AnalyticsTransactions />
    </VCol>

    <VCol cols="12" md="4">
      <AnalyticsWeeklyOverview />
    </VCol>

    <VCol cols="12" md="4">
      <AnalyticsTotalEarning />
    </VCol>

    <VCol cols="12" md="4">
      <VRow class="match-height">
        <VCol cols="12" sm="6">
          <AnalyticsTotalProfitLineCharts />
        </VCol>

        <VCol cols="12" sm="6">
          <CardStatisticsVertical v-bind="totalProfit" />
        </VCol>

        <VCol cols="12" sm="6">
          <CardStatisticsVertical v-bind="newProject" />
        </VCol>

        <VCol cols="12" sm="6">
          <AnalyticsBarCharts />
        </VCol>
      </VRow>
    </VCol>

    <VCol cols="12" md="4">
      <AnalyticsSalesByCountries />
    </VCol>

    <VCol cols="12" md="8">
      <AnalyticsDepositWithdraw />
    </VCol>

    <VCol cols="12">
      <AnalyticsUserTable />
    </VCol> -->
  </VRow>
</template>
