<script setup>
const props = defineProps({
  employee: {
    type: Object,
    default: null,
  },
})

import { ref } from 'vue'
import { useRoute } from 'vue-router'
import AddUser from './add-user.vue'
import ListUserBank from './list-user-bank.vue'

const route = useRoute()
const activeTab = ref(route.params.tab)
const emit = defineEmits(['saved', 'cancel'])

// tabs
const tabs = [
  {
    title: 'Información',
    icon: 'ri-group-line',
    tab: 'account',
  },
  {
    title: 'Cuentas',
    icon: 'ri-lock-line',
    tab: 'security',
  },

]
</script>

<template>
  <div>
    <VTabs v-model="activeTab" show-arrows class="v-tabs-pill">
      <VTab v-for="item in tabs" :key="item.icon" :value="item.tab">
        <VIcon size="20" start :icon="item.icon" />
        {{ item.title }}
      </VTab>
    </VTabs>

    <VWindow v-model="activeTab" class="mt-5 disable-tab-transition" :touch="false">
      <!-- Account -->
      <VWindowItem value="account">
        <AddUser @saved="emit('saved')" @cancel="emit('cancel')" :employee="employee" />
      </VWindowItem>

      <!-- Security -->
      <VWindowItem value="security">
        <ListUserBank @saved="emit('saved')" @cancel="emit('cancel')" :employee="employee" />
      </VWindowItem>
    </VWindow>
  </div>
</template>
