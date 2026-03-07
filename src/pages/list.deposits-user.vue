<script setup>
const props = defineProps({
  viatic: {
    type: Object,
    default: () => ({}),
  },
})

const emit = defineEmits(['close', 'cancel'])

import BaseDatatable from '@/components/BaseDatatable.vue'
import ButtonComponent from '@/components/buttonComponent.vue'
import { useSnackbar } from '@/composables/useSnackbar'
import { headersDepositsUserList } from '@/imports/headerstable'
import api from '@/services/api'
import { onMounted, ref } from 'vue'



const depositList = ref([])

const snackbar = useSnackbar()

const viaticDeposit = ref({
  id: null,
  code: '',
  oldcode: '',
})

const statusNewcode = ref(true)


const ListDeposits = async () => {
  try {
    const { data } = await api.get(`/deposits/find/${props.viatic.id}`)

    depositList.value = data.data
  } catch (err) {
    console.error(err)
    snackbar.open('Error al cargar los depósitos', 'error')
  }
}

onMounted(() => {
  ListDeposits()
})
</script>

<template>
  <div>
    <VCard>
      <VCardTitle class="d-flex justify-space-between">
        Lista de Depósitos
        <ButtonComponent icon="ri-close-circle-line" @click="emit('cancel')" />
      </VCardTitle>
      <VCardText>
        <VRow>
          <VCol cols="12">
            <BaseDatatable :items="depositList" :headers="headersDepositsUserList" class="elevation-1">
              <template #item.voucher="{ item }">
                {{ item.nro_voucher }}
              </template>

              <template #item.accounts="{ item }">
                <VChip label size="x-small">
                  ORIG: {{ item.origin_account.account_number }} - {{ item.origin_account.bank_name }}
                </VChip>
                <VChip label size="x-small">
                  DEST: {{ item.destiny_account.account_number }} - {{ item.destiny_account.bank_name }}
                </VChip>
              </template>

              <template #item.date_deposit="{ item }">
                {{ new Date(item.date_deposit).toLocaleDateString() }}
              </template>


            </BaseDatatable>
          </VCol>
        </VRow>
      </VCardText>
    </VCard>
  </div>
</template>
