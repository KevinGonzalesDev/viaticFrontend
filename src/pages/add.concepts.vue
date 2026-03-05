<script setup>
const props = defineProps({
  concept: {
    type: Object,
    default: () => ({}),
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
const conceptDescription = ref('')
const snackbar = useSnackbar()

const Formconcepts = ref({
  conceptId: '',
  description: '',
})



const addConceptFunc = async () => {
  // lógica para agregar concepto
  try {
    const payload = { ...Formconcepts.value }

    if (isEdit.value) {
      // Editar concepto
      await api.put(`/rates/concepts/`, payload)

      snackbar.open('concepto actualizado', 'success')

    } else {
      // Crear concepto
      await api.post('/rates/concepts/', payload)
      snackbar.open('concepto creado', 'success')
    }
    emit('saved')
    emit('close')
  } catch (err) {
    console.error(err)
    snackbar.open('No se pudo guardar el concepto', 'error')
  }
}

onMounted(() => {
  if (isEdit.value && props.concept) {
    Formconcepts.value.conceptId = props.concept.id
    Formconcepts.value.description = props.concept.description
  }
})
</script>

<template>
  <div>
    <VCard>
      <VCardTitle>{{ isEdit ? 'Editar Concepto' : 'Agregar Concepto' }}</VCardTitle>
      <VCardText>
        <VForm>
          <VTextField v-model="Formconcepts.description" label="Descripción del Concepto" required></VTextField>
        </VForm>
      </VCardText>
      <VCardActions>
        <VSpacer />
        <VBtn color="primary" @click="addConceptFunc">Guardar</VBtn>
        <VBtn text @click="$emit('close')">Cancelar</VBtn>
      </VCardActions>
    </VCard>
  </div>
</template>
