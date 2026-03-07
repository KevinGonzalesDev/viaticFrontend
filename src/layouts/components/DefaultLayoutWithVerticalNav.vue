<script setup>
import NavItems from '@/layouts/components/NavItems.vue'
import VerticalNavLayout from '@layouts/components/VerticalNavLayout.vue'
import logo from '../../../public/difatechpng.png'

// Components
import NavbarThemeSwitcher from '@/layouts/components/NavbarThemeSwitcher.vue'
import UserProfile from '@/layouts/components/UserProfile.vue'
import { onMounted } from 'vue'


const user = JSON.parse(localStorage.getItem('user')) || {}
const saludo = ref('')

onMounted(() => {
  const storedUser = localStorage.getItem('user')
  if (storedUser) {
    user.value = JSON.parse(storedUser)
    saludo.value = `Hola, ${user.value.name}`
  }
})
</script>


<template>
  <VerticalNavLayout>
    <!-- 👉 navbar -->
    <template #navbar="{ toggleVerticalOverlayNavActive }">
      <div class="d-flex h-100 align-center">
        <!-- 👉 Vertical nav toggle in overlay mode -->
        <IconBtn class="ms-n3 d-lg-none" @click="toggleVerticalOverlayNavActive(true)">
          <VIcon icon="ri-menu-line" />
        </IconBtn>

        <div>
          {{ saludo }}
        </div>
        <VSpacer />


        <NavbarThemeSwitcher class="me-2" />

        <UserProfile />
      </div>
    </template>

    <template #vertical-nav-header="{ toggleIsOverlayNavActive }">
      <RouterLink to="/" class="app-logo app-title-wrapper">
        <!-- eslint-disable vue/no-v-html -->
        <div class="d-flex justify-center">
          <VImg :src="logo" alt="logo" width="100" />
        </div>
        <!-- eslint-enable -->
      </RouterLink>


    </template>

    <template #vertical-nav-content>
      <NavItems />
    </template>

    <!-- 👉 Pages -->
    <slot />


  </VerticalNavLayout>
</template>

<style lang="scss" scoped>
.meta-key {
  border: thin solid rgba(var(--v-border-color), var(--v-border-opacity));
  border-radius: 6px;
  block-size: 1.5625rem;
  line-height: 1.3125rem;
  padding-block: 0.125rem;
  padding-inline: 0.25rem;
}

.app-logo {
  display: flex;
  align-items: center;
  column-gap: 0.75rem;

  .app-logo-title {
    font-size: 1.25rem;
    font-weight: 500;
    line-height: 1.75rem;
    text-transform: uppercase;
  }
}
</style>
