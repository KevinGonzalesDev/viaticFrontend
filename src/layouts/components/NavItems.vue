<script setup>
import VerticalNavSectionTitle from '@/@layouts/components/VerticalNavSectionTitle.vue';
import VerticalNavLink from '@layouts/components/VerticalNavLink.vue';

const user = JSON.parse(localStorage.getItem('user'))

const hasRole = (allowedRoles) => {
  if (!user?.roles) return false

  return allowedRoles.some(role =>
    user.roles.includes(role),
  )
}
</script>

<template>
  <!-- 👉 Apps & Pages -->
  <div>
    <VerticalNavSectionTitle :item="{
      heading: 'Analisis',
    }" />
    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria', 'employee', 'accounting'])"
      :item="{ title: 'Analitycs', icon: 'ri-home-office-line', to: '/dashboard' }" />

    <!-- vistas personales -->

    <VerticalNavSectionTitle v-if="hasRole(['admin', 'tesoreria', 'employee', 'accounting'])" :item="{
      heading: 'Vistas personales',
    }" />
    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria', 'employee', 'accounting'])"
      :item="{ title: 'Mis viaticos', icon: 'ri-list-view', to: '/list-viaticos-user' }" />

    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria', 'employee', 'accounting'])"
      :item="{ title: 'Declaraciones de Usuario', icon: 'ri-file-list-line', to: '/list-declaration-user' }" />

    <!-- vistas de gestion -->
    <VerticalNavSectionTitle v-if="hasRole(['admin', 'tesoreria', 'accounting'])" :item="{
      heading: 'Vistas de gestion',
    }" />

    <VerticalNavLink v-if="hasRole(['admin'])"
      :item="{ title: 'Usuarios', icon: 'ri-group-3-fill', to: '/list-users' }" />

    <VerticalNavLink v-if="hasRole(['admin'])"
      :item="{ title: 'Asignacion de roles', icon: 'ri-account-box-line', to: '/list-roles' }" />

    <VerticalNavLink v-if="hasRole(['admin'])"
      :item="{ title: 'Aprobacion de solicitudes', icon: 'ri-list-view', to: '/list-viaticos-adm' }" />

    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria', 'accounting'])"
      :item="{ title: 'Aprobacion de declaraciones', icon: 'ri-file-list-line', to: '/list-declaration-adm' }" />


    <VerticalNavSectionTitle v-if="hasRole(['admin', 'tesoreria', 'accounting'])" :item="{
      heading: 'Vistas contables',
    }" />
    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria', 'accounting'])"
      :item="{ title: 'Asignar presupuesto y depositos', icon: 'ri-list-view', to: '/list-viaticos-aproved' }" />

    <!-- 👉 User Interface -->
    <VerticalNavSectionTitle v-if="hasRole(['admin', 'tesoreria'])" :item="{ heading: 'Administración del sistema' }" />

    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria'])"
      :item="{ title: 'Configuración de la documentación', icon: 'ri-settings-3-line', to: '/config-doc' }" />

    <VerticalNavLink v-if="hasRole(['admin', 'tesoreria'])"
      :item="{ title: 'Costos', icon: 'ri-currency-line', to: '/list-costs' }" />

    <VerticalNavLink v-if="hasRole(['admin'])"
      :item="{ title: 'Clientes y proyectos', icon: 'ri-folder-chart-2-line', to: '/proyect' }" />


    <VerticalNavLink v-if="hasRole(['admin'])"
      :item="{ title: 'Información de la empresa', icon: 'ri-community-line', to: '/company-info' }" />
  </div>
</template>
