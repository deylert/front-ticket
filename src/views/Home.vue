<template>
  <v-snackbar class="mt-12" location="right top" :timeout="sb_timeout" :color="sb_type" elevation="24"
    :multi-line="true" vertical v-model="snackbar">
    <v-row>
      <v-col md="2">
        <v-avatar :icon="sb_icon" color="sb_type" size="40"></v-avatar>
      </v-col>
      <v-col md="10">
        <h4>{{ sb_title }}</h4>
        {{ sb_message }}
      </v-col>
    </v-row>
  </v-snackbar>
  <v-container style="min-width: 100%; min-height: 100%; background-color: #F5F5F5;">
    <v-row align="stretch">
      <!-- Información general de viajes -->
      <v-col cols="12" md="6">
        <v-row align="stretch">
          <v-col cols="12" md="6" v-for="(stat, index) in sales" :key="index">
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
                      <span v-if="stat.title !== 'Boletos Vendidos' && stat.title !== 'Incidentes'"
                        class="text-h4 font-weight-black">
                        {{ this.formatNumber(stat.value) }}
                      </span>
                      <span v-else class="text-h4 font-weight-black">
                        {{ stat.value }}
                      </span>
                    </v-col>
                    <v-col cols="12" class="text-left">
                      <span class="font-weight-black">Ver más</span>
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
          <v-col cols="12" md="12">
            <v-card class="mx-0" align="center">
              <template v-slot:title>
                <div class="d-flex align-center">
                  <v-icon color="#1976D2" class="mr-2">mdi-information</v-icon>
                  <span>Información</span>
                </div>
              </template>
              <Bar :dataArray="earlyYear" />
              <v-card-text class="py-1"> </v-card-text>
            </v-card>
          </v-col>
          <!--<v-col cols="12" md="12">
            <v-card class="mx-0" height="auto">
              <template v-slot:title>
                <v-icon color="#1976D2" left>mdi-information</v-icon>
                Información
              </template>
              <Doughnut />
              <v-card-text class="py-1"> </v-card-text>
            </v-card>
          </v-col>-->
        </v-row>
      </v-col>

      <!-- Listado de viajes recientes -->
      <v-col cols="12" md="12" class="mx-0">
        <v-card class="elevation-4">
          <v-container fluid>
            <v-toolbar color="white">
              <v-row align="center">
                <v-col cols="12" md="8" class="grow">
                  <span class="text-subtitle-1"><strong>Viajes</strong></span>
                </v-col>
              </v-row>
            </v-toolbar>
            <!--<v-card-title class="font-weight-bold;"><span color="white">Vehículos</span></v-card-title>-->
            <v-divider />
            <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
              hide-details>
            </v-text-field>
            <v-data-table :headers="headers" :search="search" :items="trips" class="elevation-1"
              :items-per-page-text="'Elementos por páginas'" items-per-page="5" no-data-text="No hay datos disponibles"
              :loading="loading" loading-text="Cargando datos...">
              <template v-slot:item.vehiclePlate="{ item }">
                <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                  <v-img :src="`${this.$axios.defaults.baseURL}images/${item.vehicleImage}?t=${Date.now()}`"
                    alt="image"></v-img>
                </v-avatar><!--+'?$'+Date.now()-->
                {{ item.vehiclePlate }}
              </template>
              <template v-slot:item.dineroGenerado="{ item }">
                <span style="font-weight: bold;">{{ this.formatNumber(item.dineroGenerado) }}</span>
              </template>
            </v-data-table>
          </v-container>
          <!--<v-list class="pa-2" density="compact">
            <v-list-item v-for="(trip, index) in microbuses" :key="index" class="py-2 border-b">
              <template v-slot:prepend>
                <v-icon color="blue-darken-2" icon="mdi-map-marker" size="32" />
              </template>

              <v-row align="center" class="flex-nowrap text-nowrap">
                <v-col cols="4" md="3">
                  <span class="text-medium-emphasis text-body-1">{{ trip.patente }}</span>
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
          </v-list>-->
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
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
export default {
  name: 'BarChart',
  components: { Bar, Doughnut, },
  data() {
    return {
      snackbar: false,
      sb_type: '',
      sb_message: '',
      sb_timeout: 2000,
      sb_title: '',
      sb_icon: '',
      role: '',
      branch_id: '',
      sales: [],
      data: {},
      earlyYear: [],
      trips: [],
      loading: false,
      type: 'Negocio',
      microbuses: [
        {
          patente: "CL-RT34",
          marca: "Mercedes-Benz Sprinter",
          capacidad: 20,
          asientosVendidos: 15,
          horario: "07:00 - 19:00",
          recorrido: "Terminal Central - Plaza Mayor",
          dineroGenerado: 15 * 800
        },
        {
          patente: "FK-YH76",
          marca: "Hyundai County",
          capacidad: 25,
          asientosVendidos: 20,
          horario: "06:30 - 18:30",
          recorrido: "Estación Norte - Universidad",
          dineroGenerado: 20 * 800
        },
        {
          patente: "GR-KL98",
          marca: "Volkswagen Crafter",
          capacidad: 19,
          asientosVendidos: 12,
          horario: "08:00 - 20:00",
          recorrido: "Aeropuerto - Centro Histórico",
          dineroGenerado: 12 * 800
        },
        {
          patente: "HH-WQ21",
          marca: "Ford Transit",
          capacidad: 22,
          asientosVendidos: 18,
          horario: "05:45 - 21:30",
          recorrido: "Barrio Industrial - Terminal Sur",
          dineroGenerado: 18 * 800
        },
        {
          patente: "JJ-ZX54",
          marca: "Iveco Daily",
          capacidad: 21,
          asientosVendidos: 19,
          horario: "07:15 - 22:00",
          recorrido: "Mercado Central - Hospital Regional",
          dineroGenerado: 19 * 800
        },
        {
          patente: "KL-PV65",
          marca: "Toyota Coaster",
          capacidad: 23,
          asientosVendidos: 20,
          horario: "06:00 - 20:45",
          recorrido: "Estación de tren - Zona Residencial",
          dineroGenerado: 20 * 800
        },
        {
          patente: "MN-RG87",
          marca: "Nissan Urvan",
          capacidad: 17,
          asientosVendidos: 14,
          horario: "07:30 - 19:45",
          recorrido: "Parque Central - Playa Norte",
          dineroGenerado: 14 * 800
        },
        {
          patente: "OP-TH98",
          marca: "Mitsubishi Fuso Rosa",
          capacidad: 24,
          asientosVendidos: 22,
          horario: "06:45 - 21:15",
          recorrido: "Terminal de buses - Universidad Técnica",
          dineroGenerado: 22 * 800
        },
        {
          patente: "QR-LM32",
          marca: "Renault Master",
          capacidad: 20,
          asientosVendidos: 17,
          horario: "05:30 - 23:00",
          recorrido: "Centro Comercial - Estación Sur",
          dineroGenerado: 17 * 800
        },
        {
          patente: "ST-XJ43",
          marca: "Peugeot Boxer",
          capacidad: 18,
          asientosVendidos: 16,
          horario: "07:00 - 18:30",
          recorrido: "Plaza Principal - Barrio Alto",
          dineroGenerado: 16 * 800
        }
      ],
      search: '',
      headers: [
        { title: 'Patente', value: 'vehiclePlate' },
        { title: 'Marca', value: 'vehicleBrand' },
        { title: 'Recorrido', value: 'route' },
        { title: 'Fecha', value: 'date' },
        { title: 'Horario', value: 'horario' },
        { title: 'Capacidad', value: 'capacidad' },
        { title: 'Asientos Vendidos', value: 'asientosVendidos' },
        { title: 'Dinero Generado', value: 'dineroGenerado' },
      ],
    };
  },
  mounted() {
    this.role = JSON.parse(LocalStorageService.getItem('role'));
    if (this.role === 'Administrador') {
      this.type = 'Negocio';
    } else {
      this.branch_id = JSON.parse(LocalStorageService.getItem('branch_id'));
      this.type = 'Sucursal';
    }
    this.initialize();
  },
  methods: {
    formatNumber(value) {
      // Si el valor es menor que 1000, devuelve el valor original con dos decimales
      if (value < 1000) {
        return (Math.round((value + Number.EPSILON) * 100) / 100).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
      }

      // Primero, redondea el valor a dos decimales
      value = Math.round((value + Number.EPSILON) * 100) / 100;

      // Convierte el valor a cadena con formato de número local (en-US)
      let formattedValue = value.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });

      return formattedValue;
    },
    month() {
      const now = new Date();
      const year = now.getFullYear();
      const month = String(now.getMonth() + 1).padStart(2, "0"); // Asegura 2 dígitos (ej. 01, 02, etc.)
      return `${year}-${month}`; // Formato YYYY-MM
    },
    async initialize() {
      this.loading = true;
      this.data = {};
      this.data.type = this.type;
      this.data.branch_id = this.branch_id;
      this.data.month = this.month();
      try {
        this.loading = true;
        const result = await handleRequest({
          endpoint: 'monthly-sales',
          method: 'POST',
          data: this.data
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.sales = result.data?.sales || [];
          this.earlyYear = result.data?.salesYear || [];
          this.trips = result.data?.trips || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.sales = [];
          this.earlyYear = [];
          this.trips = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
      } finally {
        this.loading = false;
      }
    },
    viewTripDetails(trip) {
      console.log('Detalles del viaje:', trip);
    },
    showAlert(sb_type, sb_message, sb_timeout) {
      this.sb_type = sb_type;

      if (sb_type == "success") {
        this.sb_title = "Éxito";
        this.sb_icon = "mdi-check-circle";
      }

      if (sb_type == "error") {
        this.sb_title = "Error";
        this.sb_icon = "mdi-check-circle";
      }

      if (sb_type == "warning") {
        this.sb_title = "Advertencia";
        this.sb_icon = "mdi-alert-circle";
      }
      this.sb_message = sb_message;
      this.sb_timeout = sb_timeout;
      this.snackbar = true;
    },
  },
};
</script>

<style scoped>
.text-white {
  color: white !important;
}
</style>
