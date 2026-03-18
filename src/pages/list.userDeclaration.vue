<script setup>
import BaseDatatable from '@/components/BaseDatatable.vue'
import ButtonComponent from '@/components/buttonComponent.vue'
import { headersDeclarationsUser } from '@/imports/headerstable'
import api from '@/services/api'
import { onMounted, ref } from 'vue'
import { VDateInput } from 'vuetify/labs/VDateInput'
import ViewDeclare from './view.declare.vue'

const userDeclarationsList = ref([])
const user = JSON.parse(localStorage.getItem('user')) || {}

// variables de estado para visualizacion de detallado de viatico
const showDetailForm = ref(false)
const selectedDeclarationId = ref(null)

// fuincion para abrir el detallado de viatico
const viewUserDeclarationFunc = (declaration) => {
  selectedDeclarationId.value = declaration.viatic_id
  showDetailForm.value = true
}

const closeForm = () => {
  showDetailForm.value = false
}

const filters = ref({
  startDate: null,
  endDate: null,
  status: null,
  userId: user.id
})

const anlitycInfo = ref([])

const statusList = [
  { text: 'Aprobado Tesorería', value: 'APROB_TESO' },
  { text: 'Declaracion aprobada', value: 'APROB_DEC_ADM' },
]

const statusColor = (status) => {
  switch (status) {
    case 'POSITIVE':
      return 'success'
    case 'NEGATIVE':
      return 'error'
    default:
      return 'default'
  }
}

const loadAnalyticInfo = async () => {
  try {
    const response = await api.get(`/analitycs/`, { params: { userId: user.id } })
    anlitycInfo.value = response.data.data[0]
  } catch (error) {
    console.error('Error loading analytic info:', error)
  }
}

const loadUserDeclarations = async () => {
  try {

    const params = {
      startDate: filters.value.startDate,
      endDate: filters.value.endDate,
      status: filters.value.status,
      userId: filters.value.userId
    }

    const response = await api.get(`/decviatics/user/viatics`, { params })

    userDeclarationsList.value = response.data.data
  } catch (error) {
    console.error('Error loading user declarations:', error)
  }
}

watch(filters, loadUserDeclarations, { deep: true })

const generatePDF = (id) => {
  window.open(`${import.meta.env.VITE_API_URL}/decviatics/pdf/${id}`, '_blank')
}

const generateLiquidationPDF = (id) => {
  window.open(`${import.meta.env.VITE_API_URL}/decviatics/pdf/liquidation/${id}`, '_blank')
}

const generateMobilityPDF = (id) => {
  window.open(`${import.meta.env.VITE_API_URL}/decviatics/pdf/movility/${id}`, '_blank')
}

const generateDDJJPDF = (id) => {
  window.open(`${import.meta.env.VITE_API_URL}/decviatics/pdf/ddjj/${id}`, '_blank')
}

onMounted(async () => {
  loadUserDeclarations()
  await loadAnalyticInfo()
})
</script>

<template>
  <div>
    <template v-if="!showDetailForm">
      <VRow dense>
        <VCol cols="12" md="12">
          <h2>Declaraciones de Usuario</h2>
          <p>Aquí se mostrarán las declaraciones realizadas por los usuarios.</p>
        </VCol>
        <VCol cols="12" md="4">
          <VDateInput density="compact" v-model="filters.startDate" @click:clear="filters.startDate = null"
            label="Fecha inicio" clearable />
        </VCol>

        <VCol cols="12" md="4">
          <VDateInput density="compact" v-model="filters.endDate" @click:clear="filters.endDate = null"
            label="Fecha fin" clearable />
        </VCol>

        <VCol cols="12" md="4">
          <VSelect density="compact" v-model="filters.status" :items="statusList" item-title="text" item-value="value"
            label="Estado" clearable />
        </VCol>
        <VCol cols="12">
          <VCard class="mb-4" color="primary" variant="outlined">
            <VCardTitle>
              Resumen de saldo
            </VCardTitle>
            <VCardText class="d-flex align-center">
              <VRow dense>
                <VCol cols="12">
                  <div class="d-flex flex-column ga-1 ">
                    Saldo : {{ anlitycInfo.total_balance }}
                  </div>
                </VCol>
              </VRow>
            </VCardText>
          </VCard>
        </VCol>
        <VCol cols="12">
          <BaseDatatable :headers="headersDeclarationsUser" :items="userDeclarationsList">
            <template #item.code="{ item }">
              <VChip label size="x-small">
                {{ item.viatic_code }}
              </VChip>
              <VChip v-if="item.viatic_new_code" label size="x-small" color="info">
                {{ item.viatic_new_code }}
              </VChip>
            </template>

            <template #item.project="{ item }">
              <div class="font-weight-medium">
                {{ item.proyect_name }} - {{ item.project_code }}
              </div>

              <div class="text-caption text-grey">
                {{ item.client_name }} · {{ item.location_name }}
              </div>
            </template>

            <template #item.date_data="{ item }">
              <div class="d-flex flex-column ga-1 ">
                <VChip size="small" color="secondary" label>
                  Inicio :{{ new Date(item.start_mov).toLocaleDateString() }}
                </VChip>
                <VChip size="small" color="secondary" label>
                  Fin : {{ new Date(item.end_mov).toLocaleDateString() }}
                </VChip>
                <VChip v-if="item.start_prov_date" size="small" color="info" label>
                  LLeg Prov :{{ new Date(item.start_prov_date).toLocaleDateString() }}
                </VChip>
                <VChip v-if="item.end_prov_date" size="small" color="info" label>
                  Sal Prov :{{ new Date(item.end_prov_date).toLocaleDateString() }}
                </VChip>
              </div>
            </template>



            <template #item.balance="{ item }">
              <VChip size="small" label color="success">
                Depositado: {{ item.deposit_amount }}
              </VChip>
            </template>

            <template #item.declare="{ item }">
              <VChip size="small" label class="mb-1">
                Declarado: {{ item.declare_amount }}
              </VChip>
              <br>
              <VChip size="small" label color="success">
                Aprobado: {{ item.declare_active }}
              </VChip>
              <br>
              <VChip size="small" label :color="statusColor(item.balance_amount > 0 ? 'POSITIVE' : 'NEGATIVE')">
                Saldo:
                {{ item.balance_amount }}
              </VChip>
            </template>

            <template #item.status="{ item }">
              <VChip :color="{
                'APROB_TESO': 'warning',
                'APROB_DEC_ADM': 'success',
                'APROB_ADMIN': 'success',
                'SOLICITED': 'info',
                'REFUSED': 'error',
              }[item.status]" size="small" label>
                {{ {
                  'APROB_TESO': 'Aprobado Tesorería',
                  'APROB_DEC_ADM': 'Declaración aprobada',
                  'APROB_ADMIN': 'Solicitud aprobada',
                  'SOLICITED': 'Solicitado',
                  'REFUSED': 'Rechazado',
                }[item.status]
                }}
              </VChip>

            </template>

            <template #item.actions="{ item }">
              <ButtonComponent icon="ri-hand-coin-line" tooltip="Declarar gasto" @click="viewUserDeclarationFunc(item)"
                :disabled="item.status === 'APROB_DEC_ADM'" />

              <VMenu v-if="item.status === 'APROB_DEC_ADM'">
                <template v-slot:activator="{ props }">
                  <ButtonComponent icon="ri-list-unordered" tooltip="Lista de PDFs" variant="text" v-bind="props" />
                </template>

                <VList>
                  <template v-slot:prepend>
                    <v-icon class="mr-n2" size="small"></v-icon>
                  </template>
                  <VListItem>
                    <VListItemTitle>
                      <VBtn @click="generatePDF(item.viatic_id)" prepend-icon="ri-file-pdf-2-line" variant="plain">
                        Resumen PDF
                      </VBtn>
                    </VListItemTitle>
                    <VListItemTitle>
                      <VBtn @click="generateLiquidationPDF(item.viatic_id)" prepend-icon="ri-file-pdf-2-line"
                        variant="plain">
                        Liquidación PDF
                      </VBtn>
                    </VListItemTitle>
                    <VListItemTitle>
                      <VBtn @click="generateMobilityPDF(item.viatic_id)" prepend-icon="ri-file-pdf-2-line"
                        variant="plain">
                        Movilidad PDF
                      </VBtn>
                    </VListItemTitle>
                    <VListItemTitle v-if="item.type === 'PROVINCIA'">
                      <VBtn @click="generateDDJJPDF(item.viatic_id)" prepend-icon="ri-file-pdf-2-line" variant="plain">
                        DDJJ PDF
                      </VBtn>
                    </VListItemTitle>
                  </VListItem>
                </VList>
              </VMenu>
            </template>
          </BaseDatatable>
        </VCol>
      </VRow>
    </template>
    <template v-else>
      <ViewDeclare :viatic="selectedDeclarationId" @close="closeForm" />
    </template>
  </div>
</template>
