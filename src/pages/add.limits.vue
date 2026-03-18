<script setup>
const props = defineProps({

  config: {
    type: Object,
    default: () => ({}),
  },
  selectedDistrict: {
    type: Number,
    default: null,
  },
  mode: {
    type: String,
    default: 'create',
  },
})

const emit = defineEmits(['saved', 'close'])
import { useSnackbar } from '@/composables/useSnackbar'
import api from '@/services/api'
import { computed, onMounted, ref } from 'vue'

const isEdit = computed(() => props.mode === 'edit')

const snackbar = useSnackbar()
const districts = ref([])
const categoryList = ref(['ALIMENTACION', 'ALOJAMIENTO', 'MOVILIDAD', 'COMPRAS'])

const LimitForm = ref({
  districtId: null,
  category: '',
  amount: '',
})

const loadDistricts = async provinceId => {

  try {
    const { data } = await api.get(`/locations/districts/`)

    districts.value = data.data
  } catch (err) {
    districts.value = []
    console.error('Error cargando distritos:', err)
  }
}




const addLimits = async () => {
  // lógica para agregar costo

  try {
    const payload = { ...LimitForm.value }

    if (isEdit.value) {
      // Editar costo

      await api.put('/decviatics/configurations/limits', payload)
      snackbar.open('Limite actualizado', 'success')

    } else {
      // Crear costo
      await api.post('/decviatics/configurations/limits', payload)
      snackbar.open('Limite creado', 'success')
    }
    emit('saved')
    emit('close')
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo guardar el limite', 'error')
  }
}



onMounted(() => {
  loadDistricts()

  if (isEdit.value && props.config) {
    LimitForm.value.districtId = props.config.district_id
    LimitForm.value.category = props.config.category
    LimitForm.value.amount = props.config.amount
  } else {
    LimitForm.value.districtId = props.selectedDistrict || null
  }
})
</script>

<template>
  <div>
    <VCard>
      <VCardTitle>
        <span class="text-h5">{{ isEdit ? 'Editar Limite' : 'Agregar Limite' }}</span>
      </VCardTitle>
      <VCardText>
        <VForm>
          <VRow>
            <VCol cols="12">
              <VAutocomplete v-if="!isEdit" v-model="LimitForm.districtId" label="Distrito"
                placeholder="Seleccione distrito" :items="districts" item-title="name" item-value="id">
                <template v-slot:item="{ props, item }">
                  <VListItem v-bind="props" :subtitle="item.raw.province_name" :title="item.raw.name" />
                </template>
              </VAutocomplete>
            </VCol>

            <VCol cols="12">
              <VSelect v-model="LimitForm.category" label="Concepto" placeholder="Concepto" :items="categoryList"
                item-title="description" item-value="id" />
            </VCol>

            <VCol cols="12">
              <VTextField v-model="LimitForm.amount" label="Monto" placeholder="Monto" type="number" />
            </VCol>
          </VRow>
        </VForm>
      </VCardText>

      <VCardActions class="justify-end">
        <VBtn variant="text" color="secondary" text @click="emit('close')">
          Cancelar
        </VBtn>

        <VBtn color="primary" @click="addLimits">
          Guardar limite
        </VBtn>
      </VCardActions>
    </VCard>
  </div>
</template>
