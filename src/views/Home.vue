<template>
  <v-container style="min-width: 100%; min-height: 100%; background-color: #F5F5F5;">
    <v-row align="stretch">
      <!-- Información general de viajes -->
      <v-col cols="12" md="6">
        <v-row align="stretch">
          <v-col cols="12" md="6" v-for="(stat, index) in generalStats" :key="index">
            <v-card class="mx-0" :style="{ borderRadius: '8px', border: 'none' }">
              <template v-slot:title>
                <div style="display: flex; justify-content: space-between; align-items: center; width: 100%;">
                <span>{{ stat.title }}</span> <!-- Título a la izquierda -->
                <v-avatar :color="stat.color + '-darken-2'" size="48">
                  <v-icon :color="stat.color" size="28">{{ stat.icon }}</v-icon>
                </v-avatar> <!-- Icono a la derecha -->
              </div>
              </template>
              <v-card-text>
                <v-col cols="12">
                   <v-row no-gutters>
                      <v-col cols="12" class="text-left">
                        <span class="text-h4 font-weight-black">{{ stat.value }}</span>
                      </v-col>                      
                      <v-col cols="12" class="text-left">
                        <span class="font-weight-black">Ver Más</span>
                      </v-col>
                    </v-row>
                </v-col>
              </v-card-text>
            </v-card>

          </v-col>
        </v-row>
      </v-col>

      <v-col cols="12" md="6">
        <v-row align="stretch">
          <v-col cols="12" md="6">
            <v-card class="mx-0">
              <template v-slot:title>
                <v-icon color="#1976D2" left>mdi-information</v-icon> <!-- Icono a la izquierda del título -->
                Información
              </template>
              <Bar />
              <v-card-text class="py-1"> </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card class="mx-0" height="auto">
              <template v-slot:title>
                <v-icon color="#1976D2" left>mdi-information</v-icon> <!-- Icono a la izquierda del título -->
                Información
              </template>
              <Doughnut />
              <v-card-text class="py-1"> </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>

      <!-- Listado de viajes recientes -->
      <v-col cols="12" md="12" class="mx-0">
        <v-card class="elevation-4">
          <v-card-title class="font-weight-bold;"><span color="white">Vehículos</span></v-card-title>
          <v-divider />
          <v-list class="pa-2" density="compact">
            <v-list-item v-for="(trip, index) in recentTrips" :key="index" class="py-2 border-b">
              <template v-slot:prepend>
                <v-icon color="blue-darken-2" icon="mdi-map-marker" size="32" />
              </template>

              <v-row align="center" class="flex-nowrap text-nowrap">
                <v-col cols="4" md="3">
                  <span class="text-medium-emphasis text-body-1">{{ trip.client }}</span>
                </v-col>

                <v-col cols="5" md="6">
                  <div class="d-flex align-center text-medium-emphasis">
                    <v-icon icon="mdi-car" size="18" class="mr-1" />
                    <span class="text-caption">{{ trip.vehicle }}</span>
                    <v-divider vertical class="mx-3" />
                    <v-icon icon="mdi-calendar" size="18" class="mr-1" />
                    <span class="text-caption">{{ trip.date }}</span>
                  </div>
                </v-col>

                <v-col cols="3" md="3" class="text-end">
                  <v-btn variant="outlined" size="small" color="blue-darken-2" @click="viewTripDetails(trip)">
                    Detalles
                    <v-icon icon="mdi-chevron-right" end />
                  </v-btn>
                </v-col>
              </v-row>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>

      <!-- Información de ingresos y gastos 
      <v-col cols="12" md="6" class="mx-0">
        <v-card>
          <v-card-title class="font-weight-bold" :style="{ backgroundColor: '#1976D2' }">Ingresos y Gastos</v-card-title>
          <v-divider />
          <v-row class="pa-3">
            <v-col cols="6" v-for="(finance, index) in finances" :key="index" class="text-center">
              <v-card class="elevation-3 mx-auto" :color="finance.color"
                :style="{ borderRadius: '16px', border: 'none' }">
                <v-card-item class="pa-4">
                  <template v-slot:prepend>
                    <v-avatar :color="finance.color + '-darken-2'" size="48" class="elevation-4">
                      <v-icon size="28" color="white">{{ finance.icon }}</v-icon>
                    </v-avatar>
                  </template>

                  <v-card-title class="text-h6 font-weight-bold text-white">
                    {{ finance.type }}
                  </v-card-title>
                </v-card-item>

                <v-card-text class="text-center pb-0">
                  <div class="text-h4 font-weight-black text-white">
                    ${{ finance.amount }}
                  </div>
                  <v-progress-linear v-if="finance.progress" :model-value="finance.progress" color="white" height="6"
                    class="mt-3" />
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-card>
      </v-col>-->
    </v-row>
  </v-container>
</template>

<script>
import Bar from "@/components/Bar.vue";
import Doughnut from "@/components/Doughnut.vue";
export default {
  name: 'BarChart',
  components: { Bar, Doughnut, },
  data() {
    return {
      generalStats: [
        { title: 'Boletos Vendidos', value: 320.00, color: '#1976D2', icon: 'mdi-ticket' },
        { title: 'Ingreso Generado', value: 120000.00, color: '#4CAF50', icon: 'mdi-cash-multiple' },
        { title: 'Incidentes', value: 30, color: '#F44336', icon: 'mdi-alert' },
        { title: 'Tasa de Ocupación', value: 2, color: '#FF9800', icon: 'mdi-account-group' },/*Tasa de ocupación: Promedio de pasajeros por viaje */
      ],
      recentTrips: [
        { client: 'Juan Pérez', vehicle: 'Toyota Corolla', date: '2024-12-25' },
        { client: 'María López', vehicle: 'Honda Civic', date: '2024-12-26' },
        { client: 'Carlos Sánchez', vehicle: 'Ford Fiesta', date: '2024-12-27' },
      ],
      chartData: {
        labels: ['January', 'February', 'March'],
        datasets: [{ data: [40, 20, 12] }]
      },
      chartOptions: {
        responsive: true
      },
      finances: [
        {
          type: 'Ingresos',
          amount: '15,000',
          color: 'green',
          icon: 'mdi-cash-plus',
          progress: 80
        },
        {
          type: 'Gastos',
          amount: '8,500',
          color: 'red',
          icon: 'mdi-cash-minus',
          progress: 60
        },
        {
          type: 'Beneficios',
          amount: '6,500',
          color: 'blue',
          icon: 'mdi-trending-up',
          progress: 70
        },
        {
          type: 'Pérdidas',
          amount: '2,000',
          color: 'orange',
          icon: 'mdi-trending-down',
          progress: 30
        }
      ],
    };
  },
  methods: {
    viewTripDetails(trip) {
      console.log('Detalles del viaje:', trip);
    },
  },
};
</script>

<style scoped>
.text-white {
  color: white !important;
}
</style>
