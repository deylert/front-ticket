<template>
  <v-snackbar
    class="mt-12"
    location="right top"
    :timeout="sb_timeout"
    :color="sb_type"
    elevation="24"
    :multi-line="true"
    vertical
    v-model="snackbar"
  >
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
  <v-container style="min-width: 100%">
    <v-card elevation="6" class="mx-2">
      <v-toolbar :color="paleteColors.primary">
        <v-row align="center">
          <v-col cols="12" md="8" class="grow ml-4">
            <span class="text-subtitle-1"><strong>Incidentes</strong></span>
          </v-col>
        </v-row>
      </v-toolbar>
      <v-card-text>
        <v-row>
          <v-container fluid>
            <v-cols cols="12" md="12">
              <v-row dense>
                <v-col cols="12" md="3" v-if="mostrarFila">
                  <v-autocomplete
                    :no-data-text="'No hay datos disponibles'"
                    v-model="branch_id"
                    :items="branches"
                    label="Seleccione una Sucursal"
                    prepend-inner-icon="mdi-store"
                    item-title="name"
                    item-value="id"
                    variant="underlined"
                    :rules="selectRules"
                    density="compact"
                  >
                    <template v-slot:item="{ props, item }">
                      <v-list-item
                        v-bind="props"
                        :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`"
                      >
                      </v-list-item>
                    </template> </v-autocomplete
                  ><!-- @update:model-value="initialize()">-->
                </v-col>
                <v-col cols="12" md="3">
                  <v-menu
                    v-model="menu"
                    :close-on-content-click="false"
                    :nudge-right="40"
                    transition="scale-transition"
                    offset-y
                    min-width="290px"
                  >
                    <template v-slot:activator="{ props }">
                      <v-text-field
                        v-bind="props"
                        :modelValue="dateFormatted"
                        variant="underlined"
                        prepend-icon="mdi-calendar"
                        label="Fecha"
                        density="compact"
                      ></v-text-field>
                    </template>
                    <v-locale-provider locale="es">
                      <v-date-picker
                        header="Calendario"
                        title="Seleccione la fecha"
                        :color="paleteColors.primary"
                        :modelValue="input"
                        @update:model-value="updateDate"
                        format="yyyy-MM-dd"
                      ></v-date-picker>
                    </v-locale-provider>
                  </v-menu>
                </v-col>
                <v-col cols="12" md="3">
                  <v-menu
                    v-model="menu2"
                    :close-on-content-click="false"
                    :nudge-right="40"
                    transition="scale-transition"
                    offset-y
                    min-width="290px"
                  >
                    <template v-slot:activator="{ props }">
                      <v-text-field
                        v-bind="props"
                        :modelValue="dateFormatted1"
                        variant="underlined"
                        prepend-icon="mdi-calendar"
                        label="Fecha"
                        density="compact"
                      ></v-text-field>
                    </template>
                    <v-locale-provider locale="es">
                      <v-date-picker
                        header="Calendario"
                        title="Seleccione la fecha"
                        :color="paleteColors.primary"
                        :modelValue="input2"
                        @update:model-value="updateDate1"
                        format="yyyy-MM-dd"
                        :min="dateFormatted"
                      ></v-date-picker>
                    </v-locale-provider>
                  </v-menu>
                </v-col>
                <v-col cols="12" md="2">
                  <v-btn
                    icon
                    @click="getIncidents"
                    :color="paleteColors.primary"
                    density="comfortable"
                  >
                    <v-icon>mdi-magnify</v-icon></v-btn
                  >
                </v-col>
              </v-row>
            </v-cols>
          </v-container>
        </v-row>
        <v-row dense>
          <v-col cols="12">
            <v-text-field
              class="mt-1 mb-1"
              v-model="search"
              append-icon="mdi-magnify"
              label="Buscar"
              single-line
              hide-details
            >
            </v-text-field>
            <v-data-table
              :headers="headers"
              :search="search"
              :items="incidents"
              class="elevation-1"
              style="max-height: 68vh; overflow-y: auto"
              :items-per-page-text="'Elementos por páginas'"
              no-data-text="No hay datos disponibles"
              :loading="loading"
              loading-text="Cargando datos..."
              show-expand
              @item-expanded="onItemExpanded"
            >
              <template v-slot:item.workerName="{ item }">
                <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                  <v-img
                    :src="`${this.$axios.defaults.baseURL}images/${
                      item.image
                    }?t=${Date.now()}`"
                    alt="image"
                  ></v-img> </v-avatar
                ><!--+'?$'+Date.now()-->
                {{ item.workerName }}
              </template>
              <template v-slot:item.title="{ item }">
                <div class="d-flex align-center">
                  <v-icon
                    v-if="item.title.includes('Retraso')"
                    color="warning"
                    class="mr-2"
                  >
                    mdi-clock-alert
                  </v-icon>
                  <v-icon
                    v-else-if="item.title.includes('Escaneo')"
                    color="success"
                    class="mr-2"
                  >
                    mdi-qrcode-scan
                  </v-icon>
                  <v-icon
                    v-else-if="item.title.includes('Reimpresión')"
                    color="info"
                    class="mr-2"
                  >
                    mdi-printer
                  </v-icon>
                  {{ item.title }}
                </div>
              </template>
              <!-- Columna de expansión -->
              <template v-slot:expander="{ item, isExpanded, expand }">
                <v-btn
                  size="small"
                  variant="text"
                  :color="getDetailsButtonColor(item)"
                  @click.stop="expand(!isExpanded)"
                >
                  <v-icon>{{ isExpanded ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
                  {{ isExpanded ? 'Ocultar' : 'Ver' }} detalles
                </v-btn>
              </template>

              <!-- Contenido expandido -->
              <template v-slot:expanded-row="{ columns, item }">
                <tr>
                  <td :colspan="columns.length">
                    <div class="pa-4 bg-grey-lighten-4">
                      <h4 class="text-subtitle-1 mb-2">Detalles completos:</h4>
                      
                      <!-- Icono según tipo -->
                      <div class="d-flex align-center mb-3">
                        <v-icon
                          v-if="item.title.includes('Retraso')"
                          color="warning"
                          class="mr-2"
                        >
                          mdi-clock-alert
                        </v-icon>
                        <v-icon
                          v-else-if="item.title.includes('Escaneo')"
                          color="success"
                          class="mr-2"
                        >
                          mdi-qrcode-scan
                        </v-icon>
                        <v-icon
                          v-else-if="item.title.includes('Reimpresión')"
                          color="info"
                          class="mr-2"
                        >
                          mdi-printer
                        </v-icon>
                        <strong>{{ item.title }}</strong>
                      </div>

                      <!-- Tabla de detalles con QR -->
                      <v-simple-table density="comfortable">
                        <tbody>
                          <tr v-for="(value, key) in JSON.parse(item.details)" :key="key">
                            <td class="font-weight-bold" width="200">{{ formatDetailKey(key) }}:</td>
                            <td>
                              <div v-if="key === 'qr'" :id="'qr-container-'+item.id" class="qr-container">
                                <!-- QR se generará aquí cuando se expanda -->
                              </div>
                              <template v-else>
                                {{ formatDetailValue(key, value) }}
                              </template>
                            </td>
                          </tr>
                        </tbody>
                      </v-simple-table>
                    </div>
                  </td>
                </tr>
              </template>
            </v-data-table>
          </v-col>
        </v-row>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import { paleteColors } from "@/assets/colors";
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import _ from "lodash";
import QRCode from 'qrcode';
  export default {
    data: () => ({
      snackbar: false,
      sb_type: "",
      sb_message: "",
      sb_timeout: 2000,
      sb_title: "",
      sb_icon: "",
      paleteColors: paleteColors,
      valid: true,
      loading: false,
      mostrar: false,
      dialog: false,
      dialogDelete: false,
      estimated: 0,
      timeSlotsKey: 0,
      mostrarFila: false,
      incidents: [],
      branches: [],
      data: {},
      branch_id: "",
      headers: [
        { title: "Trabajador", value: "workerName" },
        { title: "Título", value: "title" },
        { title: "Descripción", value: "description" },
        { title: 'Acciones', key: 'data-table-expand' }
      ],
      editedItem: {
        startDate: null,
        endDate: null,
        branch_id: "",
      },
      originalItem: {
        startDate: null,
        endDate: null,
        branch_id: "",
      },
      defaultItem: {
        startDate: null,
        endDate: null,
        branch_id: "",
      },
      editedIndex: -1,
      search: "",
      menu: false,
      menu2: false,
      input: null,
      input2: null,
      tab: null,
      nameRules: [
        (v) => !!v || "El campo es requerido",
        (v) => (v && v.length <= 50) || "El campo debe tener menos de 51 caracteres",
        (v) => (v && v.length >= 3) || "El campo debe tener al menos 3 caracteres",
      ],
      selectRules: [(v) => !!v || "Seleccionar al menos un elemento"],
      priceRules: [
        (v) => !!v || "El precio es obligatorio", // El campo es obligatorio
        (v) =>
          /^[0-9]+(\.[0-9]{1,2})?$/.test(v) ||
          "El precio debe ser un número válido con hasta 2 decimales", // Valida el formato del precio
        (v) => v > 0 || "El precio debe ser un número positivo", // El precio debe ser positivo
      ],
    }),
    computed: {
      dateFormatted() {
        const date = this.input ? new Date(this.input) : new Date();
        return date.toISOString().split("T")[0];
      },
      getDate() {
        return this.input ? new Date(this.input) : new Date();
      },
      dateFormatted1() {
        const date = this.input2 ? new Date(this.input2) : new Date();
        return date.toISOString().split("T")[0];
      },
      getDate1() {
        return this.input2 ? new Date(this.input2) : new Date();
      },
    },
    mounted() {
      this.role = JSON.parse(LocalStorageService.getItem("role"));
      if (this.role === "Administrador") {
        this.showBranches();
      } else {
        this.branch_id = LocalStorageService.getItem("branch_id");
        this.initialize();
      }
    },

    methods: {
      async showBranches() {
        try {
          const result = await handleRequest({
            endpoint: "branch",
            method: "GET",
          });

          if (result.success) {
            // Si la solicitud es exitosa, asignamos las sucursales
            this.branches = result.data?.branches || [];
            this.editedItem.branch_id = this.branches[0].id;
            this.branch_id = this.branches[0].id;
          } else {
            // Si no hay datos, asignamos un array vacío
            this.branches = [];
            this.mostrarFila = false;
          }
        } catch (error) {
          this.mostrarFila = false;
          // Captura de errores no controlados
          this.showAlert(
            "error",
            "Ocurrió un error inesperado al procesar la solicitud.",
            3000
          );
        } finally {
          this.mostrarFila = true;
          this.loading = false;
          this.initialize();
        }
      },
    updateDate(val) {
      this.input = val;
      this.editedItem.startDate = this.dateFormatted;
      this.menu = false;
    },
    updateDate1(val) {
      this.input2 = val;
      this.editedItem.endDate = this.dateFormatted1;
      this.menu2 = false;
    },
    async initialize() {
      this.data = {};
      this.data.branch_id = this.branch_id;
      const today = new Date();
      const formattedDate = today.toISOString().split("T")[0]; // Formato: YYYY-MM-DD
      //this.data.date = formattedDate;
      try {
        this.loading = true;
        const result = await handleRequest({
          endpoint: "incident-date",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.incidents = result.data?.incidents || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.incidents = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al procesar la solicitud.",
          3000
        );
      } finally {
        this.loading = false;
      }
    },
    async getIncidents() {
      this.data = {};
      this.data.branch_id = this.branch_id;
      this.data.startDate =
        this.editedItem.startDate ?? new Date().toISOString().split("T")[0];
      this.data.endDate =
        this.editedItem.endDate ?? new Date().toISOString().split("T")[0];
      try {
        this.loading = true;
        const result = await handleRequest({
          endpoint: "incident-date",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.incidents = result.data?.incidents || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.incidents = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al procesar la solicitud.",
          3000
        );
      } finally {
        this.loading = false;
      }
    },
    
    getDetailsButtonColor(item) {
      if (item.title.includes('Reimpresión')) return 'info';
      if (item.title.includes('Retraso')) return 'warning';
      if (item.title.includes('Escaneo')) return 'success';
      return 'primary';
    },
    formatDetailKey(key) {
      const keysMap = {
        qr: "Código QR",
        action: "Acción",
        ticket_id: "ID Ticket",
        new_status: "Nuevo Estado",
        previous_status: "Estado Anterior",
        arrival: "Llegada Programada",
        actualEnd: "Llegada Real",
        difference: "Diferencia",
        actualStart: "Inicio Real",
        scheduledStart: "Inicio Programado",
        print: "Número de Reimpresión",
        method: "Método de Pago",
        quantity: "Cantidad",
        price: "Precio Unitario",
        total: "Total",
        trip_id: "ID del Viaje"
      };
      return keysMap[key] || key.split('_').map(w => w.charAt(0).toUpperCase() + w.slice(1)).join(' ');
    },
    formatDetailValue(key, value) {
      if (key === 'qr') return '';
      if (key === 'difference' && typeof value === 'number') {
        const hours = Math.floor(value / 60);
        const minutes = value % 60;
        return `${hours}h ${minutes}m`;
      }
      if (key === 'method') {
        const methods = {
          cash: 'Efectivo',
          card: 'Tarjeta',
          transfer: 'Transferencia'
        };
        return methods[value] || value;
      }
      if (['price', 'total'].includes(key)) {
        return new Intl.NumberFormat('es-MX', {
          style: 'currency',
          currency: 'MXN'
        }).format(value);
      }
      return value;
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
    async onItemExpanded({ item, value }) {
      // Solo generar QR cuando se expande y tiene datos QR
      if (value && item.details?.qr) {
        console.log('item', item);
        await this.$nextTick();
        await this.generateQRCode(item);
      }
    },
    
    async generateQRCode(item) {
      if (!item.details?.qr) return;
      console.log('item', item);
      const containerId = `qr-container-${item.id}`;
      await this.$nextTick();
      
      const container = document.getElementById(containerId);
      if (!container) return;
      
      // Limpiar contenedor por si ya tenía un QR
      container.innerHTML = '';
      
      const canvas = document.createElement('canvas');
      container.appendChild(canvas);
      
      try {
        await QRCode.toCanvas(canvas, item.details.qr, {
          width: 120,
          margin: 1,
          color: {
            dark: '#000000',
            light: '#ffffff'
          }
        });
      } catch (error) {
        console.error('Error generando QR:', error);
        container.innerHTML = '<span class="error-text">Error al generar código QR</span>';
      }
    },
  },
};
</script>
<style scoped>
.selected-tab {
  background-color: #1976d2;
  /* Fondo del tab seleccionado */
  color: white;
  /* Texto blanco */
  border-radius: 4px;
  /* Esquinas redondeadas, opcional */
}
.v-data-table >>> .v-data-table__expanded__content {
  box-shadow: inset 0 0 8px rgba(0,0,0,0.1);
}
</style>
