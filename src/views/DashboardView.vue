<template>
  <v-app class="fill-height">
    <v-layout class="rounded rounded-md" style="height: 100vh; display: flex; flex-direction: row; max-width: 100%; overflow-x: hidden;">
      <!-- Barra de navegación superior -->
      <AppBarView app style="flex-shrink: 0;"></AppBarView>

      <!-- Contenido principal con navegación lateral -->
      <NavigationDrawerView app class="pt-12" style="flex-shrink: 0;"></NavigationDrawerView>

      <v-main class="mt-1" style="max-width: 100vw; overflow-x: hidden;">
          <router-view />
      </v-main>

      <!-- Pie de página -->
      <v-footer app>
        <!-- Contenido del footer -->
      </v-footer>
    </v-layout>
  </v-app>
</template>

<script>
import NavigationDrawerView from './NavigationDrawerView.vue';
import AppBarView from '@/layouts/default/AppBar.vue';

export default {
  components: {
    NavigationDrawerView,
    AppBarView,
  },
};
</script>

<style scoped>
.fill-height {
  min-height: 100vh;
}
</style>
