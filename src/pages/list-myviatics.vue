<script setup>
import BaseDatatable from '@/components/BaseDatatable.vue'
import ButtonComponent from '@/components/buttonComponent.vue'
import ConfirmDialog from '@/components/modalConfirmation.vue'
import { useSnackbar } from '@/composables/useSnackbar'
import { headerviaticsUser } from '@/imports/headerstable'
import api from '@/services/api'
import { onMounted, ref } from 'vue'
import AddViatic from './add-viatic.vue'
import ListDepositsUser from './list.deposits-user.vue'



const viaticModal = ref(false)
const selectedViatic = ref(null)
const mode = ref('create')

// estados de confirmacion
const showConfirm = ref(false)
const viaticIdToDelete = ref(null)
const snackbar = useSnackbar()

// estados de vista de depositos
const depositModal = ref(false)
const selectedDeposit = ref(null)


const viaticList = ref([])
const user = ref()
const search = ref('')

const openListDepositsModal = (viatic) => {
  selectedViatic.value = viatic
  depositModal.value = true
}

const openNewViaticModal = () => {
  mode.value = 'create'
  selectedViatic.value = null
  viaticModal.value = true
}

const openEditViaticModal = async viatic => {
  // Lógica para editar el viático
  mode.value = 'edit'
  selectedViatic.value = viatic
  viaticModal.value = true
}

const openDeleteViaticConfirm = (viaticId) => {
  viaticIdToDelete.value = viaticId
  showConfirm.value = true
}

const getBudgetStatus = (item) => {
  const deposited = Number(item.deposit_amount || 0)
  const budget = Number(item.budget_total || 0)

  if (item.status !== 'APROB_TESO') return null

  if (deposited === 0) {
    return { text: 'PRESUPUESTADO', color: 'info' }
  }

  if (deposited < budget) {
    return { text: 'ABONADO', color: 'error' }
  }

  return { text: 'ABONADO', color: 'success' }
}

const closeViaticModal = () => {
  viaticModal.value = false
  selectedViatic.value = null
}


const ListViaticsbyId = async () => {
  try {
    const { data } = await api.get('/viatics/listviaticbyid/' + user.value.id)

    viaticList.value = data.data
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo cargar los viáticos', 'error')
  }
}

const deleteViatic = async () => {
  try {
    await api.delete('/viatics/deleteviatic/' + viaticIdToDelete.value)
    snackbar.open('Viático eliminado correctamente', 'success')
    ListViaticsbyId()
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo eliminar el viático', 'error')
  }
}

onMounted(() => {
  const storedUser = localStorage.getItem('user')
  if (storedUser) {
    user.value = JSON.parse(storedUser)
  }
  ListViaticsbyId()
})
</script>

<template>
  <div>
    <v-dialog v-model="viaticModal" max-width="800" scrollable>
      <AddViatic :mode="mode" :viatic="selectedViatic" @close="closeViaticModal" @saved="ListViaticsbyId" />
    </v-dialog>

    <VDialog v-model="depositModal" max-width="800" scrollable>
      <ListDepositsUser :viatic="selectedViatic" @close="ListViaticsbyId" @cancel="depositModal = false" />
    </VDialog>

    <ConfirmDialog v-model="showConfirm" title="Eliminar solicitud de viático"
      message="Esta acción no se puede deshacer. ¿Deseas continuar?" confirm-text="Sí, eliminar" cancel-text="No"
      @confirm="deleteViatic" />

    <h1>Listado de Viáticos</h1>
    <!-- Aquí va el contenido del listado de viáticos -->
    <VRow>
      <VCol cols="12" class="d-flex justify-end align-center">
        <VBtn color="primary" @click="openNewViaticModal">Solicitar viatico</VBtn>
      </VCol>
      <VCol cols="12">
        <!-- {{ viaticList }} viáticos encontrados -->
        <BaseDatatable v-model:search="search" :items="viaticList" :headers="headerviaticsUser">

          <template #item.employee_name="{ item }">
            {{ item.user_name }} {{ item.user_lastname }}
          </template>

          <template #item.project_data="{ item }">
            <div class="font-weight-medium">
              {{ item.project_name }}
            </div>

            <div class="text-caption text-grey">
              {{ item.client_name }} · {{ item.location_name }}
            </div>

          </template>

          <template #item.date_details="{ item }">
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

          <template #item.status="{ item }">
            <VChip :color="{
              'APROB_TESO': 'warning',
              'APROB_DEC_ADM': 'success',
              'APROB_ADMIN': 'system',
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
            <ButtonComponent v-if="item.status === 'SOLICITED'" tooltip="Editar solicitud" icon="ri-edit-line"
              @click="openEditViaticModal(item)" />
            <ButtonComponent v-if="item.status != 'SOLICITED'" icon="ri-wallet-line" tooltip="Ver mis depositos"
              @click="openListDepositsModal(item)" />

            <ButtonComponent v-if="item.status === 'SOLICITED'" icon="ri-delete-bin-line" tooltip="Eliminar viatico"
              @click="openDeleteViaticConfirm(item.id)" />
          </template>
        </BaseDatatable>
      </VCol>
    </VRow>
  </div>
</template>
