<script setup>
const props = defineProps({
  account: {
    type: Object,
    default: () => ({}),
  },

  companyId: {
    type: [String, Number],
    required: true,
  },

  mode: {
    type: String,
    default: 'create',
  },
})

const emit = defineEmits(['saved', 'close'])
import { useSnackbar } from '@/composables/useSnackbar.js'
import { required } from '@/imports/rulesImport.js'
import api from '@/services/api'
import { computed, onMounted, ref } from 'vue'



const banklist = ['BCP', 'BBVA', 'INTERBANK', 'SCOTIABANK', 'BANCO DE LA NACION', 'BANCO PICHINCHA']
const snackbar = useSnackbar()
const isEdit = computed(() => props.mode === 'edit')

const BankAccountForm = ref({
  ownerType: 'Empresa',
  ownerId: '',
  bankName: '',
  accountNumber: '',
  cci: '',
  currency: '',
  status: true,
  bankId: null,
})

const addAccountFunc = async () => {
  // lógica para agregar cuenta bancaria
  try {
    const payload = { ...BankAccountForm.value }

    if (isEdit.value) {
      // Editar cuenta bancaria
      await api.put(`/bank/`, payload)
      snackbar.open('Cuenta bancaria actualizada exitosamente', 'success')
    } else {
      // Crear cuenta bancaria
      await api.post('/bank/', payload)
      snackbar.open('Cuenta bancaria creada exitosamente', 'success')
    }
    emit('saved')
    emit('close')
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo guardar la cuenta bancaria', 'error')
  }
}

onMounted(() => {
  if (props.mode === 'edit') {
    BankAccountForm.value = {
      ownerType: props.account.owner_type,
      ownerId: props.account.owner_id,
      bankName: props.account.bank_name,
      accountNumber: props.account.account_number,
      cci: props.account.cci,
      currency: props.account.currency,
      status: props.account.status,
      bankId: props.account.id || null,
    }
  } else {
    BankAccountForm.value.ownerId = props.companyId
  }
})
</script>

<template>
  <div>
    <VCard>
      <VCardTitle>
        {{ isEdit ? 'Editar cuenta bancaria' : 'Agregar cuenta bancaria' }}
      </VCardTitle>
      <VCardText>
        <VRow>
          <VCol cols="12">
            <VSelect v-model="BankAccountForm.ownerType" :items="['Empresa', 'Empleado']" label="Tipo de propietario"
              :rules="[required]" readonly />
          </VCol>

          <VCol cols="12">
            <VSelect v-model="BankAccountForm.bankName" :items="banklist" label="Seleccionar banco"
              :rules="[required]" />
          </VCol>

          <VCol cols="12">
            <VTextField v-model="BankAccountForm.accountNumber" :rules="[required]" label="Número de cuenta" />
          </VCol>

          <VCol cols="12">
            <VTextField v-model="BankAccountForm.cci" :rules="[required]" label="CCI" />
          </VCol>

          <VCol cols="12">
            <VSelect v-model="BankAccountForm.currency" :rules="[required]" :items="['SOLES', 'DOLARES']"
              label="Moneda" />
          </VCol>
        </VRow>
      </VCardText>
      <VCardActions>
        <VSpacer />
        <VBtn text @click="$emit('close')">Cancelar</VBtn>

        <VBtn color="primary" @click="addAccountFunc">
          {{ isEdit ? 'Guardar cambios' : 'Crear cuenta bancaria' }}
        </VBtn>
      </VCardActions>
    </VCard>
  </div>
</template>
