<script setup>
import ButtonComponent from '@/components/buttonComponent.vue';
import api from '@/services/api';
import { onMounted, ref } from 'vue';
import AddConfig from './add.config.vue';
import AddLimits from './add.limits.vue';

const listConfigurations = ref([])
const listLimits = ref([])

// variables de estado para el modal de configuración de opciones de documentación
const showConfigModal = ref(false)
const selectedConfig = ref(null)
const modeConfigModal = ref('create')
const selectedtype = ref(null)


const showLimitModal = ref(false)
const selectedLimit = ref(null)
const modeLimitModal = ref('create')
const selectedDistrict = ref(null)


const search = ref('')
const page = ref(1)
const itemsPerPage = ref(4)




const loadLiquidityConfigs = async () => {
  try {
    const { data } = await api.get(`/decviatics/configurations/liquidation`)

    listConfigurations.value = data.data

  } catch (error) {
    console.error('Error fetching liquidation configurations:', error)
  }
}

const loadLimits = async () => {
  try {
    const { data } = await api.get(`/decviatics/configurations/limits`)

    listLimits.value = data.data

  } catch (error) {
    console.error('Error fetching limits:', error)
  }
}

const createConfig = (documentType) => {
  modeConfigModal.value = 'create'
  selectedConfig.value = null
  selectedtype.value = documentType
  showConfigModal.value = true
}

const editConfig = (item) => {
  modeConfigModal.value = 'edit'
  selectedConfig.value = item
  showConfigModal.value = true
}

const createLimit = (district) => {
  modeLimitModal.value = 'create'
  selectedLimit.value = null
  selectedDistrict.value = district
  showLimitModal.value = true
}

const editLimit = (item) => {
  modeLimitModal.value = 'edit'
  selectedLimit.value = item
  selectedDistrict.value = item.district_id
  showLimitModal.value = true
}

onMounted(() => {
  loadLiquidityConfigs()
  loadLimits()
})
</script>

<template>
  <div>
    <VRow>
      <VCol cols="12" md="12">
        <VCard>
          <VCardTitle class="d-flex justify-space-between">
            Configuración de opciones de documentación
            <VBtn @click="createConfig()">
              Agregar opción
            </VBtn>
          </VCardTitle>
          <VCardText>

            <VDataIterator :items="listConfigurations" v-model:page="page" :items-per-page="itemsPerPage"
              :search="search">
              <template #default="{ items }">
                <VRow>
                  <VCol v-for="docconfig in items" :key="docconfig.document_type" cols="12" md="6">
                    <VCard>
                      <!-- Header del distrito -->
                      <VCardTitle class="d-flex justify-space-between align-center">
                        <span class="text-h6">
                          {{ docconfig.raw.document_type }}
                        </span>

                        <VBtn size="small" color="primary" prepend-icon="ri-add-line"
                          @click="createConfig(docconfig.raw.document_type)">
                          Agregar
                        </VBtn>
                      </VCardTitle>

                      <VDivider />

                      <!-- Lista de presupuestos -->
                      <VList density="compact">
                        <VListItem v-for="conf in docconfig.raw.documentary" :key="conf.id">
                          <VListItemTitle>
                            {{ conf.label }}
                            <VChip size="small" label class="ml-2">
                              {{ conf.category }}
                            </VChip>
                          </VListItemTitle>



                          <template #append>
                            <ButtonComponent :icon="conf.active ? 'ri-toggle-fill' : 'ri-toggle-line'"
                              :tooltip="conf.active ? 'Desactivar configuración' : 'Activar configuración'"
                              @click="desactivateConfig(conf.id, conf.active)" />

                            <ButtonComponent icon="ri-pencil-line" tooltip="Editar configuración" color="primary"
                              @click="editConfig(conf)" />


                          </template>
                        </VListItem>
                      </VList>
                    </VCard>
                  </VCol>
                </VRow>
              </template>

              <template #footer="{ pageCount }">
                <div class="d-flex justify-center mt-4">
                  <VPagination v-model="page" :length="pageCount" size="small" rounded />
                </div>
              </template>
            </VDataIterator>
          </VCardText>
        </VCard>
      </VCol>
      <VCol cols="12">
        <VCard>
          <VCardTitle class="d-flex justify-space-between">
            Lista de limites de gasto por destino
            <VBtn @click="createLimit()">
              Agregar limite
            </VBtn>
          </VCardTitle>
          <VCardSubtitle>
            Estos limites se aplican para cada gasto registrado en el sistema por dia, dependiendo del distrito
            seleccionado
            en el gasto y la categoria asociada al gasto.
          </VCardSubtitle>
          <VCardText>
            <VDataIterator :items="listLimits" v-model:page="page" :items-per-page="itemsPerPage" :search="search">
              <template #default="{ items }">
                <VRow>
                  <VCol v-for="limConfig in items" :key="limConfig.district_id" cols="12" md="3">
                    <VCard>
                      <!-- Header del distrito -->
                      <VCardTitle class="d-flex justify-space-between align-center">
                        <span class="text-h6">
                          {{ limConfig.raw.district_name }}
                        </span>

                        <VBtn size="small" color="primary" prepend-icon="ri-add-line"
                          @click="createLimit(limConfig.raw.district_id)">
                          Agregar
                        </VBtn>
                      </VCardTitle>

                      <VDivider />

                      <!-- Lista de presupuestos -->
                      <VList density="compact">
                        <VListItem v-for="lim in limConfig.raw.limits" :key="lim.id">
                          <VListItemTitle>
                            {{ lim.category }}
                            <VChip size="small" label class="ml-2">
                              {{ lim.amount }}
                            </VChip>
                          </VListItemTitle>

                          <template #append>
                            <ButtonComponent icon="ri-pencil-line" tooltip="Editar configuración" color="primary"
                              @click="editLimit(lim)" />

                          </template>
                        </VListItem>
                      </VList>
                    </VCard>
                  </VCol>
                </VRow>
              </template>

              <template #footer="{ pageCount }">
                <div class="d-flex justify-center mt-4">
                  <VPagination v-model="page" :length="pageCount" size="small" rounded />
                </div>
              </template>
            </VDataIterator>
          </VCardText>
        </VCard>
      </VCol>

    </VRow>
    <VDialog v-model="showConfigModal" max-width="500px">
      <AddConfig :config="selectedConfig" :selectedtype="selectedtype" :mode="modeConfigModal"
        @close="showConfigModal = false" @saved="loadLiquidityConfigs" />
    </VDialog>

    <VDialog v-model="showLimitModal" max-width="500px">
      <AddLimits :config="selectedLimit" :selectedDistrict="selectedDistrict" :mode="modeLimitModal"
        @close="showLimitModal = false" @saved="loadLiquidityConfigs" />
    </VDialog>
  </div>
</template>
