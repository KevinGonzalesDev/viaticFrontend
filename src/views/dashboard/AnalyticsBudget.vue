<script setup>
import api from '@/services/api'
import { onMounted, ref } from 'vue'
const listSaldo = ref([])

const storedUser = localStorage.getItem('user')
const user = storedUser ? JSON.parse(storedUser) : null

const loadSaldo = async () => {
  if (!user) return
  try {
    const response = await api.get(`/analitycs/`, { params: { userId: user.id } })
    listSaldo.value = response.data.data[0]
  } catch (error) {
    console.error('Error loading saldo:', error)
  }
}

onMounted(async () => {
  await loadSaldo()
})
</script>
<template>
  <VCard>
    <VCardTitle class="d-flex justify-space-between">
      Saldo actual

      <h6 class="text-h6">
        <a href="javascript:void(0)" class="text-primary">
          {{ listSaldo.total_balance }}
        </a>
      </h6>
    </VCardTitle>
    <VCardSubtitle>
      Detallado de saldos por viaticos por sanear
    </VCardSubtitle>

    <VCardText>
      <VList class="card-list">
        <VListItem v-for="list in listSaldo.debt_viatics" :key="list.id">
          <template #prepend>
            <div class="me-4">
              <VChip size="x-small" variant="outlined">
                {{ list.code }}
              </VChip>
            </div>
          </template>

          <VListItemTitle class="font-weight-medium mb-1">
            Depositado : {{ list.deposit_amount }}
          </VListItemTitle>
          <VListItemSubtitle class="text-body-1">
            Declarado : {{ list.declared_amount }}
          </VListItemSubtitle>

          <template #append>
            <VListItemAction class="text-success font-weight-medium">
              {{ list.balance }}
            </VListItemAction>
          </template>
        </VListItem>
      </VList>
    </VCardText>
  </VCard>
</template>
