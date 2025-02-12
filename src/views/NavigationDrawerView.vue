<template>
  <v-card>
    <v-layout>
      <v-navigation-drawer permanent class="pt-4 my-1" color="white" theme="dark" app>
        <!--<template v-slot:prepend>
    <v-list-item
      color="#1976D2"
      lines="two"
      variant="flat"
      :style="{ backgroundColor: '#1976D2', display: 'flex', justifyContent: 'center', alignItems: 'center' }"
    >
    <template v-slot:title>
      <v-icon class="mb-3" size="45">mdi-bus</v-icon>-->
        <!--<div style="display: flex; align-items: center; gap: 0;">
        <span style="font-size: 2rem; font-weight: bold; color: white;">
          Dashboard
        </span>
      </div>
    </template>
</v-list-item>
</template>-->
        <template v-slot:prepend>
          <v-list-item class="text-subtitle-1" lines="two" variant="flat"
            :prepend-avatar="`${this.$axios.defaults.baseURL}images/${this.imageBranch}`" :title="this.title"
            :subtitle="this.subtitle"
            :style="{ backgroundColor: '#ECEFF1', display: 'flex', justifyContent: 'center', alignItems: 'center', color: '#000000' }">
          </v-list-item>
        </template>
        <!--prepend-avatar=`${this.$axios.defaults.baseURL}images/${imageBusiness}`-->
        <v-divider></v-divider>

        <v-list density="compact" nav>
          <v-list-item prepend-icon="mdi-view-dashboard-outline" title="Dashboard" to="home" value="home"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-store" title="Negocio" to="company" value="company"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-store" title="Sucursales" to="branch" value="branch"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-devices" title="Dsipositivos" to="device" value="device"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-bus" title="Vehículos" to="vehicle" value="vehicle"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-account" title="Trabajadores" to="worker" value="worker"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-map-marker" title="Lugares" to="location" value="location"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-road-variant" title="Rutas" to="route" value="route"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-steering" title="Viajes" to="trip" value="trip"
            class="list-item"></v-list-item>
          <v-list-item prepend-icon="mdi-ticket" title="Tickets" to="tick" value="tick" class="list-item"></v-list-item>


          <v-list-group value="Mantenedores">
            <template v-slot:activator="{ props }">
              <v-list-item v-bind="props" prepend-icon="mdi-progress-wrench" title="Mantenedores"></v-list-item>
            </template>


            <v-list-item v-for="([title, icon, to], i) in mainteiners" :key="i" :title="title" :prepend-icon="icon"
              :value="title" :to="to" style="padding-left: 20px !important"></v-list-item>
          </v-list-group>
        </v-list>

        <!-- Espaciador -->
        <v-spacer></v-spacer>
        <template v-slot:append>

          <!-- Opciones en la parte inferior -->
          <v-list density="compact" nav>
            <v-list-item prepend-icon="mdi-cog-outline" title="Configuración" to="/settings" value="settings"
              class="list-item"></v-list-item>
            <!--<v-list-item prepend-icon="mdi-earth" title="Idioma" to="/language" value="language" class="list-item"></v-list-item>-->
          </v-list>
        </template>
      </v-navigation-drawer>
      <v-main style="height: 94vh"></v-main>
    </v-layout>
  </v-card>
</template>

<script>
import LocalStorageService from "@/LocalStorageService";
export default {
  data: () => ({
    open: ['Users'],
    mainteiners: [
      //['Categorías', 'mdi-text-box-outline', '/category'],
      //['Prioridades', 'mdi-star-circle-outline', '/priority'],
      ['Roles', 'mdi-account-cog-outline', '/role'],
      ['Permisos', 'mdi-shield-check', '/permission'],
    ],
    title: '',
    imageBranch: '',
    subtitle: 'Negocio',
    role: '',
  }),
  mounted() {
    /*this.name = JSON.parse(LocalStorageService.getItem('name'));
    this.user = JSON.parse(LocalStorageService.getItem('user'));
    this.user_id = JSON.parse(LocalStorageService.getItem('user_id'));
    this.rol_id = LocalStorageService.getItem('role_id');*/
    this.role = JSON.parse(LocalStorageService.getItem('role'));
    if (this.role === 'Administrador') {
      this.title = JSON.parse(LocalStorageService.getItem('nameBusiness'));
      this.imageBranch = LocalStorageService.getItem('imageBusiness').replace(/['"]+/g, '');
    } else {
      this.title = JSON.parse(LocalStorageService.getItem('nameBranch'));
      this.imageBranch = LocalStorageService.getItem('imageBranch').replace(/['"]+/g, '');
      this.subtitle = 'Sucursal'
    }
  },
}
</script>
<style>
.list-item {
  padding: 2px 4px !important;
  /* Reduce el padding vertical y horizontal */
}

.maintainer-item {
  padding-left: 16px !important;
  /* Indentación menor para subelementos */
}
</style>
