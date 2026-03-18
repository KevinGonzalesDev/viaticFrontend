<script setup>
const props = defineProps({
  viatic: {
    type: Number,
    default: null,
  },
})

const emit = defineEmits(['loaded', 'close'])

import BaseDatatable from '@/components/BaseDatatable.vue'
import ButtonComponent from '@/components/buttonComponent.vue'
import { useSnackbar } from '@/composables/useSnackbar.js'
import {
  headersDDJJAlimUser,
  headersDDJJMovUser,
  headersLiquidationsUser,
  headersMovilityUser,
} from '@/imports/headerstable'
import api from '@/services/api'
import { onMounted, ref } from 'vue'

const snackbar = useSnackbar()
const viaticDetails = ref(null)
const viaticItemsLiquidation = ref([])
const viaticItemsMovility = ref([])
const viaticItemsAliDeclaration = ref([])
const viaticItemsMovDeclaration = ref([])
const listLimits = ref([])

const showDeclareModal = ref(false)
const modeDeclareModal = ref('create') // 'create' o 'edit'
const selectedDeclareViatic = ref(null)
const selectedViatic = ref(null)

const declareIdToDelete = ref(null)
const showConfirm = ref(false)

const Dateformsatatus = ref(false)

const formDate = ref({
  fechaSalida: null,
  fechaLlegada: null,
})


const viaticStatus = ref(null)

const itemsStatus = [
  { text: 'Aprobado Tesorería', value: 'APROB_TESO' },
  { text: 'Declaracion aprobada', value: 'APROB_DEC_ADM' },
]

const ImageDialog = ref(false)
const selectedImage = ref(null)

const viewDocument = (item) => {
  if (!item.document_url) {
    snackbar.open('No hay documento disponible', 'error')
    return
  }

  const url = import.meta.env.VITE_APP_URL + item.document_url
  window.open(url, '_blank')
}

const viewImage = (item) => {
  if (!item.image_url) {
    snackbar.open('No hay imagen disponible', 'error')
    return
  }

  selectedImage.value = import.meta.env.VITE_APP_URL + item.image_url
  ImageDialog.value = true
}


const loadViaticDetails = async () => {
  try {
    const { data } = await api.get(`/decviatics/detail/${props.viatic}`)

    viaticDetails.value = data.data

  } catch (error) {
    console.error('Error loading viatic details:', error)
  }
}

const loadLimitDistric = async () => {
  try {
    const { data } = await api.get(`/decviatics/configurations/limits/district/${viaticDetails.value.district_id}`)

    listLimits.value = data.data

  } catch (error) {
    console.error('Error loading limit district:', error)
  }
}

const loadViaticItems = async () => {
  try {

    const { data } = await api.get(`/decviatics/items/${props.viatic}`)
    const items = data.data

    const dailyTotals = calculateDailyLimits(items)


    const limitsMap = Object.fromEntries(
      listLimits.value.map(l => [l.category, Number(l.amount)])
    )

    // console.log(limitsMap);

    items.forEach(item => {

      const date = item.expense_real_date.slice(0, 10)
      const category = item.optionobject?.category

      const total = dailyTotals?.[date]?.[category] || 0
      const limit = limitsMap[category]

      item.limitExceeded = limit && total > limit

      const expenseDate = new Date(item.expense_real_date).setHours(0, 0, 0, 0)
      const startDate = new Date(viaticDetails.value.viatic_start).setHours(0, 0, 0, 0)
      const endDate = new Date(viaticDetails.value.viatic_end).setHours(0, 0, 0, 0)



      item.limitDateExceeded = expenseDate < startDate || expenseDate > endDate

    })

    viaticItemsLiquidation.value = items.filter(i => i.document_type === 'LIQUIDACION')
    viaticItemsMovility.value = items.filter(i => i.document_type === 'MOVILIDAD')

    viaticItemsAliDeclaration.value =
      items.filter(i =>
        i.document_type === 'DECLARACION_JURADA' &&
        i.optionobject.category === 'ALIMENTACION'
      )

    viaticItemsMovDeclaration.value =
      items.filter(i =>
        i.document_type === 'DECLARACION_JURADA' &&
        i.optionobject.category === 'MOVILIDAD'
      )

  } catch (error) {
    console.error('Error loading viatic items:', error)
  }
}

const calculateDailyLimits = (items) => {

  const totals = {}



  items.forEach(item => {

    if (!item.is_active) return

    const date = item.expense_real_date.slice(0, 10)
    const category = item.optionobject?.category

    if (!category) return

    if (!totals[date]) totals[date] = {}
    if (!totals[date][category]) totals[date][category] = 0

    totals[date][category] += Number(item.amount)

  })

  return totals
}

const totalLiquidation = computed(() => {
  return viaticItemsLiquidation.value.reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalActiveLiquidation = computed(() => {
  return viaticItemsLiquidation.value
    .filter(item => item.is_active)
    .reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalMovilidad = computed(() => {
  return viaticItemsMovility.value.reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalActiveMovilidad = computed(() => {
  return viaticItemsMovility.value
    .filter(item => item.is_active)
    .reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalDdjj = computed(() => {
  return [
    ...viaticItemsAliDeclaration.value,
    ...viaticItemsMovDeclaration.value
  ].reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalActiveDdjj = computed(() => {
  return [
    ...viaticItemsAliDeclaration.value.filter(item => item.is_active),
    ...viaticItemsMovDeclaration.value.filter(item => item.is_active)
  ].reduce((acc, item) => acc + Number(item.amount), 0)
})

const totalActiveGeneral = computed(() => {
  return (
    totalActiveLiquidation.value +
    totalActiveMovilidad.value +
    totalActiveDdjj.value
  )
})



const totalGeneral = computed(() => {
  return (
    totalLiquidation.value +
    totalMovilidad.value +
    totalDdjj.value
  )
})

const desactivateDeclaration = async (id, active) => {
  try {
    await api.put(`/decviatics/admin/desactivate/${id}`, { active: !active })

    snackbar.open(`Gasto ${!active ? 'activado' : 'desactivado'} con exito`, 'success')
    await loadViaticItems()
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo actualizar el estado del gasto', 'error')
  }
}

const updateViaticStatus = async () => {
  try {
    await api.put(`/decviatics/status/${props.viatic}`, { status: viaticStatus.value })

    snackbar.open('Estado actualizado con exito', 'success')
    await loadViaticDetails()
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo actualizar el estado del viaje', 'error')
  }
}


onMounted(async () => {
  await loadViaticDetails()
  await loadLimitDistric()
  await loadViaticItems()
  viaticStatus.value = viaticDetails.value?.viatic_status || null
})
</script>

<template>
  <div>
    <VRow>
      <VCol cols="12" class="d-flex justify-start">
        <VBtn prepend-icon="ri-arrow-left-circle-fill" color="primary" @click="emit('close')">
          Volver
        </VBtn>
      </VCol>
      <VCol cols="12">
        <div class="text-h5 text-center font-weight-medium">
          Detalles de la Declaración
        </div>
      </VCol>

      <!-- INFORMACION GENERAL -->
      <VCol cols="12" md="12">
        <VRow>

          <!-- INFORMACION DEL VIAJE -->
          <VCol cols="12">
            <VCard>
              <VCardTitle class="text-center">Información del viaje</VCardTitle>

              <VCardText class="text-body-2">
                <VRow>
                  <VCol cols="12" md="8">
                    <VCard variant="outlined">
                      <VCardText class="text-body-2">

                        <div><strong>N° VIAJE:</strong> {{ viaticDetails?.viatic_code }}</div>
                        <div><strong>PROYECTO:</strong> {{ viaticDetails?.cost_center }}</div>
                        <div><strong>TRABAJADOR:</strong> {{ viaticDetails?.user_name }} {{ viaticDetails?.user_lastname
                          }}
                        </div>
                        <div><strong>DNI:</strong> {{ viaticDetails?.user_dni }}</div>
                        <div><strong>AREA:</strong> {{ viaticDetails?.user_area }}</div>
                        <div><strong>MOTIVO:</strong> {{ viaticDetails?.proyect_name }}</div>
                        <div><strong>LUGAR:</strong> {{ viaticDetails?.location_name }}</div>
                      </VCardText>
                    </VCard>
                  </VCol>
                  <VCol cols="12" md="4">
                    <VRow>
                      <VCol cols="12">
                        <div class="d-flex flex-column ga-1 ">
                          <VChip size="small" color="secondary" label>
                            Inicio :{{ new Date(viaticDetails?.viatic_start).toLocaleDateString() }}
                          </VChip>
                          <VChip size="small" color="secondary" label>
                            Fin : {{ new Date(viaticDetails?.viatic_end).toLocaleDateString() }}
                          </VChip>
                          <VChip v-if="viaticDetails?.start_prov_date" size="small" color="info" label>
                            LLeg Prov :{{ new Date(viaticDetails?.start_prov_date).toLocaleDateString() }}
                          </VChip>
                          <VChip v-if="viaticDetails?.end_prov_date" size="small" color="info" label>
                            Sal Prov :{{ new Date(viaticDetails?.end_prov_date).toLocaleDateString() }}
                          </VChip>
                        </div>
                      </VCol>
                      <VCol cols="12" class="d-flex">
                        <VSelect density="compact" v-model="viaticStatus" :items="itemsStatus" item-title="text"
                          item-value="value" label="Estado" />

                        <ButtonComponent icon="ri-save-line" tooltip="Guardar estado" color="primary"
                          @click="updateViaticStatus" />
                      </VCol>
                    </VRow>
                  </VCol>
                </VRow>

              </VCardText>

            </VCard>
          </VCol>




        </VRow>
      </VCol>


      <!-- CAMBIO DE ESTADO -->



      <!-- RESUMENES -->
      <VCol cols="12">
        <VRow>

          <!-- DECLARADOS -->
          <VCol cols="12" md="6">
            <VCard>
              <VCardTitle>Gastos declarados</VCardTitle>

              <VCardText class="text-body-2">
                <div><strong>Liquidación:</strong> S/ {{ totalLiquidation }}</div>
                <div><strong>Movilidad:</strong> S/ {{ totalMovilidad }}</div>
                <div><strong>Declaración jurada:</strong> S/ {{ totalDdjj }}</div>

                <VDivider class="my-2" />

                <div class="text-subtitle-2">
                  <strong>Total:</strong> S/ {{ totalGeneral }}
                </div>
              </VCardText>

            </VCard>
          </VCol>

          <!-- APROBADOS -->
          <VCol cols="12" md="6">
            <VCard>
              <VCardTitle>Gastos aprobados</VCardTitle>

              <VCardText class="text-body-2">
                <div><strong>Liquidación:</strong> S/ {{ totalActiveLiquidation }}</div>
                <div><strong>Movilidad:</strong> S/ {{ totalActiveMovilidad }}</div>
                <div><strong>Declaración jurada:</strong> S/ {{ totalActiveDdjj }}</div>

                <VDivider class="my-2" />

                <div class="text-subtitle-2">
                  <strong>Total:</strong> S/ {{ totalActiveGeneral }}
                </div>
              </VCardText>

            </VCard>
          </VCol>

        </VRow>
      </VCol>

      <VCol cols="12">
        <VCard>
          <VCardText>
            <BaseDatatable tabletitle="LIQUIDACION" :headers="headersLiquidationsUser" :items="viaticItemsLiquidation"
              dontfooter>
              <template #item.option_name="{ item }">
                {{ item.optionobject ? item.optionobject.label : '' }}
              </template>
              <template #item.expense_date="{ item }">
                <VChip size="x-small" :color="item.limitDateExceeded ? 'error' : ''">
                  {{ item.expense_real_date ? new Date(item.expense_real_date).toLocaleDateString() : '' }}
                </VChip>
              </template>

              <template #item.amount="{ item }">
                <VChip size="x-small" :color="item.limitExceeded ? 'error' : ''">
                  S/ {{ item.amount }}
                </VChip>
              </template>

              <template #item.active="{ item }">
                <VChip :color="item.is_active ? 'primary' : 'grey'">
                  {{ item.is_active ? 'Activo' : 'Inactivo' }}
                </VChip>
              </template>

              <template #item.actions="{ item }">

                <ButtonComponent icon="ri-file-pdf-line" tooltip="Ver factura"
                  :color="item.document_url ? 'primary' : 'grey'" @click="viewDocument(item)" />

                <ButtonComponent icon="ri-image-line" tooltip="Ver comprobante"
                  :color="item.image_url ? 'primary' : 'grey'" @click="viewImage(item)" />

                <ButtonComponent :icon="item.is_active ? 'ri-toggle-fill' : 'ri-toggle-line'"
                  :tooltip="item.is_active ? 'Declaracion aprobada' : 'Declaracion desactivada'"
                  @click="desactivateDeclaration(item.id, item.is_active)" />

              </template>
            </BaseDatatable>
          </VCardText>
        </VCard>
      </VCol>

      <VCol cols="12">
        <VCard>
          <VCardText>
            <BaseDatatable tabletitle="MOVILIDAD LIMA" :headers="headersMovilityUser" :items="viaticItemsMovility"
              dontfooter>
              <!-- Aquí puedes agregar los detalles adicionales de la declaración -->
              <template #item.option_name="{ item }">
                {{ item.optionobject ? item.optionobject.label : '' }}
              </template>

              <template #item.expense_date="{ item }">
                <VChip size="x-small" :color="item.limitDateExceeded ? 'error' : ''">
                  {{ item.expense_real_date ? new Date(item.expense_real_date).toLocaleDateString() : '' }}
                </VChip>
              </template>

              <template #item.amount="{ item }">
                <VChip size="x-small" :color="item.limitExceeded ? 'error' : ''">
                  S/ {{ item.amount }}
                </VChip>
              </template>
              <template #item.active="{ item }">
                <VChip :color="item.is_active ? 'primary' : 'grey'">
                  {{ item.is_active ? 'Activo' : 'Inactivo' }}
                </VChip>
              </template>

              <template #item.actions="{ item }">

                <ButtonComponent icon="ri-file-pdf-line" tooltip="Ver factura"
                  :color="item.document_url ? 'primary' : 'grey'" @click="viewDocument(item)" />

                <ButtonComponent icon="ri-image-line" tooltip="Ver comprobante"
                  :color="item.image_url ? 'primary' : 'grey'" @click="viewImage(item)" />

                <ButtonComponent :icon="item.is_active ? 'ri-toggle-fill' : 'ri-toggle-line'"
                  :tooltip="item.is_active ? 'Declaracion aprobada' : 'Declaracion desactivada'"
                  @click="desactivateDeclaration(item.id, item.is_active)" />

              </template>
            </BaseDatatable>
          </VCardText>
        </VCard>
      </VCol>
      <VCol v-if="viaticDetails?.type != 'LIMA'" cols="12">
        <VCard>
          <VCardTitle class="d-flex justify-space-between">
            DECLARACION JURADA
          </VCardTitle>
          <VCardText>
            <BaseDatatable tabletitle="ALIMENTACION" :headers="headersDDJJAlimUser" :items="viaticItemsAliDeclaration"
              dontfooter>
              <!-- Aquí puedes agregar los detalles adicionales de la declaración -->
              <template #item.option_name="{ item }">
                {{ item.optionobject ? item.optionobject.label : '' }}
              </template>

              <template #item.expense_date="{ item }">
                <VChip size="x-small" :color="item.limitDateExceeded ? 'error' : ''">
                  {{ item.expense_real_date ? new Date(item.expense_real_date).toLocaleDateString() : '' }}
                </VChip>
              </template>
              <template #item.amount="{ item }">
                <VChip size="x-small" :color="item.limitExceeded ? 'error' : ''">
                  S/ {{ item.amount }}
                </VChip>
              </template>
              <template #item.active="{ item }">
                <VChip :color="item.is_active ? 'primary' : 'grey'">
                  {{ item.is_active ? 'Activo' : 'Inactivo' }}
                </VChip>
              </template>
              <template #item.actions="{ item }">

                <ButtonComponent icon="ri-file-pdf-line" tooltip="Ver factura"
                  :color="item.document_url ? 'primary' : 'grey'" @click="viewDocument(item)" />

                <ButtonComponent icon="ri-image-line" tooltip="Ver comprobante"
                  :color="item.image_url ? 'primary' : 'grey'" @click="viewImage(item)" />

                <ButtonComponent :icon="item.is_active ? 'ri-toggle-fill' : 'ri-toggle-line'"
                  :tooltip="item.is_active ? 'Declaracion aprobada' : 'Declaracion desactivada'"
                  @click="desactivateDeclaration(item.id, item.is_active)" />
              </template>
            </BaseDatatable>

            <BaseDatatable tabletitle="MOVILIDAD" :headers="headersDDJJMovUser" :items="viaticItemsMovDeclaration">
              <!-- Aquí puedes agregar los detalles adicionales de la declaración -->
              <template #item.expense_date="{ item }">
                <VChip size="x-small" :color="item.limitDateExceeded ? 'error' : ''">
                  {{ item.expense_real_date ? new Date(item.expense_real_date).toLocaleDateString() : '' }}
                </VChip>
              </template>

              <template #item.option_data="{ item }">
                <VChip label size="x-small">
                  {{ item.movility_method }}
                </VChip>
                {{ item.optionobject ? item.optionobject.label : '' }}
              </template>

              <template #item.amount="{ item }">
                <VChip size="x-small" :color="item.limitExceeded ? 'error' : ''">
                  S/ {{ item.amount }}
                </VChip>
              </template>

              <template #item.active="{ item }">
                <VChip :color="item.is_active ? 'primary' : 'grey'">
                  {{ item.is_active ? 'Activo' : 'Inactivo' }}
                </VChip>
              </template>

              <template #item.actions="{ item }">


                <ButtonComponent icon="ri-file-pdf-line" tooltip="Ver factura"
                  :color="item.document_url ? 'primary' : 'grey'" @click="viewDocument(item)" />

                <ButtonComponent icon="ri-image-line" tooltip="Ver comprobante"
                  :color="item.image_url ? 'primary' : 'grey'" @click="viewImage(item)" />

                <ButtonComponent :icon="item.is_active ? 'ri-toggle-fill' : 'ri-toggle-line'"
                  :tooltip="item.is_active ? 'Declaracion aprobada' : 'Declaracion desactivada'"
                  @click="desactivateDeclaration(item.id, item.is_active)" />

              </template>
            </BaseDatatable>
          </VCardText>
        </VCard>
      </VCol>
    </VRow>
    <VDialog v-model="ImageDialog" max-width="600">
      <VCard>
        <VCardTitle>Comprobante</VCardTitle>

        <VCardText class="text-center">
          <img :src="selectedImage" style="max-width: 100%; border-radius: 8px;" />
        </VCardText>

        <VCardActions>
          <VSpacer />
          <VBtn text @click="ImageDialog = false">Cerrar</VBtn>
        </VCardActions>
      </VCard>
    </VDialog>
  </div>
</template>
