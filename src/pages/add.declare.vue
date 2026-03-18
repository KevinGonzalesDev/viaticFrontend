<script setup>
const props = defineProps({
  viatic: {
    type: Number,
    default: null,
  },
  item: {
    type: Object,
    default: () => ({}),
  },
  details: {
    type: Object,
    default: () => ({}),
  },
  mode: {
    type: String,
    default: 'create',
  },
})

const emit = defineEmits(['saved', 'close'])
import ButtonComponent from '@/components/buttonComponent.vue'
import { useSnackbar } from '@/composables/useSnackbar'
import api from '@/services/api'
import { computed, onMounted, ref, watch } from 'vue'
import { VDateInput } from 'vuetify/labs/VDateInput'


const isEdit = computed(() => props.mode === 'edit')
const snackbar = useSnackbar()



const Formdeclare = ref({
  viaticId: props.viatic.viatic_id,
  declareId: null,
  documentType: 'LIQUIDACION',
  optionObject: null,
  optionId: null,
  documentNumber: '',
  travelFrom: '',
  travelTo: '',
  expenseRealDate: new Date(),
  expenseDate: new Date(),
  amount: '',
  category: '',
  paymentMethod: 'EFECTIVO',
  movilityMethod: null,
  DocumentFile: null,
  imageFile: null,
  removeDocument: false,
  removeImage: false
})

const typesDocument = ref([''])
const paymentMethods = ['EFECTIVO', 'TARJETA', 'TRANSFERENCIA']
const mobilityMethods = ['TAXI', 'MOTOTAXI', 'COLECTIVO', 'BUS']
const listOptions = ref([])
const existingItem = ref(null)
const ImageDialog = ref(false)
const selectedImage = ref(null)
const details = computed(() => props.details || {})

const getOpcionswithType = async type => {
  try {
    Formdeclare.value.description = ''

    const { data } = await api.get(`/decviatics/configurations/liquidation/type/${type}`)

    listOptions.value = data.data
    snackbar.open('Opciones cargadas', 'success')
  } catch (error) {
    console.error('Error fetching options:', error)
    snackbar.open('No se pudieron cargar las opciones', 'error')
  }
}

const addDepositViatic = async () => {
  try {

    const formatDate = (date) => {
      if (!date) return ''
      return new Date(date).toISOString()
    }

    const formData = new FormData()
    const form = Formdeclare.value

    // Campos normales
    formData.append('declareId', form.declareId)
    formData.append('viaticId', form.viaticId)
    formData.append('documentType', form.documentType)
    // ⚠️ optionObject (importante)
    formData.append('optionObject', JSON.stringify(form.optionObject))
    formData.append('documentNumber', form.documentNumber || '')
    formData.append('travelFrom', form.travelFrom || '')
    formData.append('travelTo', form.travelTo || '')
    formData.append('expenseDate', formatDate(form.expenseDate))
    formData.append('expenseRealDate', formatDate(form.expenseRealDate))
    formData.append('amount', form.amount)
    formData.append('movilityMethod', form.movilityMethod || '')
    formData.append('paymentMethod', form.paymentMethod)
    formData.append('removeDocument', form.removeDocument || false)
    formData.append('removeImage', form.removeImage || false)


    const pdf = Array.isArray(form.DocumentFile) ? form.DocumentFile[0] : form.DocumentFile
    const image = Array.isArray(form.imageFile) ? form.imageFile[0] : form.imageFile

    if (pdf) {
      formData.append('document', pdf)
    }

    if (image) {
      formData.append('image', image)
    }

    if (isEdit.value) {
      await api.put(`/decviatics/expenses/`, formData)
    } else {
      await api.post('/decviatics/expenses/', formData)
    }

    snackbar.value = { show: true, message: 'Gasto guardado', color: 'success' }
    emit('saved')
    emit('close')

  } catch (err) {
    console.error(err)
    snackbar.value = { show: true, message: 'Error al guardar', color: 'error' }
  }
}

const normalizeDate = (date) => {
  return new Date(date).toISOString().split('T')[0]
}

const isOutOfRange = (date) => {
  if (!date || !details.value.viatic_start || !details.value.viatic_end)
    return false

  const selected = normalizeDate(date)
  const start = normalizeDate(details.value.viatic_start)
  const end = normalizeDate(details.value.viatic_end)




  return selected < start || selected > end
}

const isMovilidad = computed(() => {
  return Formdeclare.value.documentType !== 'LIQUIDACION' &&
    Formdeclare.value.optionObject?.category === 'MOVILIDAD'
})

const isMovilidadddj = computed(() => {
  return Formdeclare.value.documentType === 'DECLARACION_JURADA' &&
    Formdeclare.value.optionObject?.category === 'MOVILIDAD'
})

const handleDocumentTypeChange = async (newType) => {
  Formdeclare.value.optionObject = null
  await getOpcionswithType(newType)
}

const removePdf = () => {
  Formdeclare.value.DocumentFile = null
  Formdeclare.value.removeDocument = true
}

const removeImageFile = () => {
  Formdeclare.value.imageFile = null
  Formdeclare.value.removeImage = true
}

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

watch(isMovilidad, newVal => {
  if (!newVal) {
    Formdeclare.value.travelFrom = ''
    Formdeclare.value.travelTo = ''
  }
})

watch(() => Formdeclare.value.documentType, newType => {
  getOpcionswithType(newType)
})



const optionId = computed(() => Formdeclare.value.optionObject?.id ?? null)



onMounted(async () => {
  await getOpcionswithType(Formdeclare.value.documentType)

  if (props.viatic.type === 'LIMA') {
    typesDocument.value = ['LIQUIDACION', 'MOVILIDAD']
  } else {
    typesDocument.value = ['LIQUIDACION', 'MOVILIDAD', 'DECLARACION_JURADA']
  }

  if (isEdit.value) {
    existingItem.value = props.item

    Object.assign(Formdeclare.value, {
      declareId: props.item.id,
      viaticId: props.viatic.viatic_id,
      documentType: props.item.document_type,
      documentNumber: props.item.document_number,
      travelFrom: props.item.travel_from,
      travelTo: props.item.travel_to,
      expenseDate: props.item.expense_date ? new Date(props.item.expense_date) : new Date(),
      expenseRealDate: props.item.expense_real_date ? new Date(props.item.expense_real_date) : new Date(),
      amount: props.item.amount,
      category: props.item.category,
      paymentMethod: props.item.payment_method,
      movilityMethod: props.item.movility_methods,
      optionObject: props.item.optionobject,


      DocumentFile: null,
      imageFile: null,

      removeDocument: false,
      removeImage: false // No podemos cargar la imagen, solo indicamos que existe
    })

  }
})
</script>

<template>
  <div>
    <VCard>
      <VCardTitle>
        {{ isEdit ? 'EDITAR GASTO' : 'AGREGAR GASTO' }}
      </VCardTitle>
      <VCardText>
        <VRow>

          <VCol cols="12" sm="6">
            <VAutocomplete v-model="Formdeclare.documentType" :items="typesDocument" label="TIPO"
              @update:model-value="handleDocumentTypeChange" outlined dense />
          </VCol>
          <VCol cols="12" sm="12">
            <VAutocomplete v-model="Formdeclare.optionObject" :items="listOptions" label="Opciones" outlined dense
              item-value="id" item-title="label" return-object>
              <template v-slot:item="{ props, item }">
                <VListItem v-bind="props">
                  <VListItemSubtitle class="text-caption">
                    {{ item.raw.category }}
                  </VListItemSubtitle>
                </VListItem>
              </template>

              <template #selection="{ item }">
                {{ item.raw.label }}
                <VChip size="x-small" color="primary">
                  {{ item.raw.category }}
                </VChip>
              </template>
            </VAutocomplete>
          </VCol>
          <VCol v-if="Formdeclare.documentType === 'LIQUIDACION'" cols="12" sm="6">
            <VTextField v-model="Formdeclare.documentNumber" label="Número de Documento" outlined dense />
          </VCol>

          <VCol v-if="isMovilidad" cols="12" sm="6">
            <VTextField v-model="Formdeclare.travelFrom" label="Desde" outlined dense />
          </VCol>

          <VCol v-if="isMovilidad" cols="12" sm="6">
            <VTextField v-model="Formdeclare.travelTo" label="Hasta" outlined dense />
          </VCol>
          <VCol cols="12" sm="6">
            <VDateInput v-model="Formdeclare.expenseRealDate" :error="isOutOfRange(Formdeclare.expenseRealDate)"
              :error-messages="isOutOfRange(Formdeclare.expenseRealDate)
                ? 'La fecha está fuera del rango del viático'
                : ''" label="Fecha de Gasto" prepend-icon="" variant="outlined" />
          </VCol>

          <VCol v-if="Formdeclare.documentType === 'LIQUIDACION'" cols="12" sm="6">
            <VDateInput v-model="Formdeclare.expenseDate" :error="isOutOfRange(Formdeclare.expenseDate)"
              :error-messages="isOutOfRange(Formdeclare.expenseDate)
                ? 'La fecha está fuera del rango del viático'
                : ''" label="Fecha de facturacion" variant="outlined" />
          </VCol>
          <VCol cols="12" sm="6">
            <VTextField v-model="Formdeclare.amount" label="Monto" type="number" outlined dense />
          </VCol>
          <VCol v-if="isMovilidadddj" cols="12" sm="6">
            <VSelect v-model="Formdeclare.movilityMethod" label="Método de movilidad" :items="mobilityMethods" outlined
              dense />
          </VCol>
          <VCol cols="12" sm="6">
            <VSelect v-model="Formdeclare.paymentMethod" label="Método de Pago" :items="paymentMethods" outlined
              dense />
          </VCol>
          <VCol v-if="Formdeclare.documentType === 'LIQUIDACION' && !existingItem?.document_url" cols="12" sm="6">
            <VFileInput v-model="Formdeclare.DocumentFile" label="Subir PDF" accept="application/pdf"
              prepend-icon="mdi-file-pdf-box" show-size />
          </VCol>



          <VCol v-if="!existingItem?.image_url" cols="12" sm="6">
            <VFileInput v-model="Formdeclare.imageFile" label="Subir imagen o tomar foto" accept="image/*"
              capture="environment" prepend-icon="mdi-camera" show-size />
          </VCol>

          <VCol v-if="isEdit && existingItem?.document_url" cols="12">
            <div class="d-flex justify-space-between">
              <VChip size="small" label :color="Formdeclare.removeDocument ? 'error' : 'success'">
                📄 Documento cargado </VChip>
              <div>
                <ButtonComponent icon="ri-eye-line" @click="viewDocument(existingItem)" />
                <ButtonComponent icon="ri-delete-bin-line" color="error" @click="removePdf" />
              </div>
            </div>
          </VCol>

          <VCol v-if="isEdit && existingItem?.image_url" cols="12">
            <div class="d-flex justify-space-between">
              <VChip size="small" label :color="Formdeclare.removeImage ? 'error' : 'success'">
                🖼️ Imagen cargada
              </VChip>

              <div>
                <ButtonComponent icon="ri-eye-line" @click="viewImage(existingItem)" />
                <ButtonComponent icon="ri-delete-bin-line" color="error" @click="removeImageFile" />
              </div>
            </div>
          </VCol>

        </VRow>
      </VCardText>
      <VCardActions>
        <VSpacer />
        <VBtn text @click="emit('close')">Cancelar</VBtn>
        <VBtn color="primary" @click="addDepositViatic">
          {{ isEdit ? 'Guardar gasto' : 'Agregar gasto' }}
        </VBtn>
      </VCardActions>
    </VCard>

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
