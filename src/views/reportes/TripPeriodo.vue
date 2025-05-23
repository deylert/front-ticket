<template>
  <v-container style="min-width: 100%; min-height: 100%">
    <v-card elevation="6" class="mx-2">
      <v-toolbar :color="paleteColors.primary">
        <v-row align="center">
          <v-col cols="12" md="8" class="grow ml-4">
            <span class="text-subtitle-1"><strong>Ventas Diarias</strong></span>
          </v-col>
          <v-col cols="12" md="3" class="text-right">
            <v-btn
              class="text-subtitle-1 ml-12"
              :color="paleteColors.white"
              variant="tonal"
              elevation="2"
              prepend-icon="mdi-file-excel-box"
              @click="exportToExcel"
            >
              Exportar a Excel
            </v-btn>
          </v-col>
        </v-row>
      </v-toolbar>
      <v-card-text>
        <v-row>
          <v-col cols="12" md="2">
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
                  prepend-inner-icon="mdi-calendar"
                  label="Fecha de inicio"
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
          <v-col cols="12" md="2">
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
                  prepend-inner-icon="mdi-calendar"
                  label="Fecha Terminación"
                  density="compact"
                ></v-text-field>
              </template>
              <v-locale-provider locale="es">
                <v-date-picker
                  header="Calendario"
                  title="Seleccione la fecha"
                  :color="paleteColors.primary"
                  :modelValue="input2"
                  format="yyyy-MM-dd"
                  :min="dateFormatted"
                  @update:model-value="updateDate1"
                ></v-date-picker
                ><!--@update:model-value="updateDate2"-->
              </v-locale-provider>
            </v-menu>
          </v-col>
          <v-col cols="12" md="3" v-if="type === 'Sucursal' && mostrarFila">
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
          <v-col cols="12" md="2">
            <v-select
              v-model="type"
              :items="options"
              label="Seleccione una opción"
              variant="underlined"
              density="compact"
              item-title="title"
              item-value="value"
            >
              <!-- Personalizar cómo se muestran las opciones en la lista -->
              <template v-slot:item="{ props, item }">
                <v-list-item v-bind="props">
                  <template v-slot:prepend>
                    <v-icon :icon="item.raw.icon"></v-icon>
                    <!-- Ícono de la opción -->
                  </template>
                </v-list-item>
              </template>
              <!-- Ícono para el select -->
              <template v-slot:prepend-inner>
                <v-icon icon="mdi-form-dropdown"></v-icon>
              </template>
            </v-select>
          </v-col>
          <v-col cols="12" md="3">
            <v-btn
              icon
              @click="initialize"
              :color="paleteColors.primary"
              density="comfortable"
            >
              <v-icon>mdi-magnify</v-icon></v-btn
            >
          </v-col>
        </v-row>
  
            <!--<v-card-text>
                            <v-col cols="12" class="pa-0"> 
                                <v-row class="ma-0"> 
                                    <v-col cols="12" class="text-h6 text-center pa-1"> 
                                        {{ response.nombre }}
                                    </v-col>
                                    <v-col cols="12" class="text-center pa-1">
                                        FECHA: {{ this.response.fecha }}
                                    </v-col>
                                    <v-col cols="12" class="pa-1">
                                        <strong>RESUMEN:</strong>
                                    </v-col>
                                    <v-col cols="12" class="pa-1">
                                        <strong>EMISIÓN DE PASAJES:</strong>
                                    </v-col>
                                    <v-col cols="12" class="pa-1">
                                        <strong>Pasajes emitidos:</strong> {{ this.response.pasajesEmitidos }}
                                    </v-col>
                                    <v-col cols="12" class="pa-1">
                                        <strong>Reimpresiones:</strong> {{ this.response.reimpresiones }}
                                    </v-col>
                                    <v-col cols="12" v-for="(total, index) in this.response.totalesPorMetodo"
                                        :key="index" class="pa-1">
                                        {{ total.metodo }}: {{ (Number(total.cantidad)) }}
                                    </v-col>
                                    <v-col cols="12" class="pa-1">
                                        <strong>TOTALES:</strong>
                                    </v-col>
                                    <v-col cols="12" v-for="(total, index) in this.response.totalesPorMetodo"
                                        :key="index" class="pa-1">
                                        <strong>{{ total.metodo }}:</strong> ${{ this.formatNumber(Number(total.total)) }}
                                    </v-col>
                                    <v-col cols="12" class="font-weight-bold pa-1">
                                        TOTAL: ${{ this.formatNumber(Number(this.response.totales)) }}
                                    </v-col>
                                    <v-col cols="12" class="font-weight-bold pa-1">
                                        ---------------------------------------------
                                        <br>
                                        TRAMOS:
                                        <br>
                                        ---------------------------------------------
                                    </v-col>
                                    <v-col cols="12" v-for="(tramo, index) in this.response.tramos" :key="index"
                                        class="pa-1">
                                        <v-col cols="12" class="font-weight-bold pa-1">
                                            {{ tramo.nombre }}
                                        </v-col>
                                        <v-col cols="12" class="pa-1">
                                            Total Pasajes: {{ tramo.totalPasajes }}
                                        </v-col>
                                        <v-col cols="12" v-for="(total, index) in tramo.totalesPorMetodo" :key="index"
                                            class="pa-1">
                                            {{ total.metodo }}: {{ (Number(total.cantidad)) }}
                                        </v-col>
                                        <v-col cols="12" class="pa-1">
                                            Total Tramo: {{ this.formatNumber(Number(tramo.totalTramo)) }}
                                        </v-col>
                                        <v-col cols="12" v-for="(total, index) in tramo.totalesPorMetodo" :key="index"
                                            class="pa-1">
                                            {{ total.metodo }}: ${{ this.formatNumber(Number(total.total)) }}
                                        </v-col>
                                        <v-col cols="12" class="font-weight-bold pa-1">
                                            ---------------------------------------------
                                        </v-col>
                                    </v-col>
                                </v-row>
                            </v-col>
                        </v-card-text>-->
                    <v-row class="ma-0">
                       <v-col cols="12" class="text-h6 text-center pa-1">
                                        {{ response.nombre }}
                                    </v-col>
                                    
                                    <!-- Fecha -->
                                    <v-col cols="12" class="text-center pa-1">
                                        FECHA: {{ response.fecha }}
                                    </v-col>
                                    
                                    <!-- Sección RESUMEN -->
                                    <v-col cols="12" class="pa-1">
                                        <strong>RESUMEN:</strong>
                                    </v-col>
                                    
                                    <!-- Cards informativas -->
                                    <v-col cols="12" md="4" class="pa-1">
                                        <v-card class="h-100" title="Pasajes emitidos" :subtitle="response.pasajesEmitidos">
                                        <template v-slot:prepend>
                                            <v-avatar color="blue-lighten-1">
                                            <v-icon>mdi-ticket-confirmation</v-icon>
                                            </v-avatar>
                                        </template>
                                        <template v-slot:append>
                                            <v-avatar color="blue-lighten-4" size="32">
                                            <v-icon color="blue-darken-2" size="20">mdi-plus</v-icon>
                                            </v-avatar>
                                        </template>
                                        </v-card>
                                    </v-col>
                                    
                                    <v-col cols="12" md="4" class="pa-1">
                                        <v-card class="h-100" title="Reimpresiones" :subtitle="response.reimpresiones">
                                        <template v-slot:prepend>
                                            <v-avatar color="orange-lighten-1">
                                            <v-icon>mdi-printer</v-icon>
                                            </v-avatar>
                                        </template>
                                        <template v-slot:append>
                                            <v-avatar color="orange-lighten-4" size="32">
                                            <v-icon color="orange-darken-2" size="20">mdi-refresh</v-icon>
                                            </v-avatar>
                                        </template>
                                        </v-card>
                                    </v-col>
                                    <v-col cols="12" md="4" class="pa-1">
                                    <v-card class="h-100" title="TOTAL GENERAL" :subtitle="'$' + formatNumber(Number(response.totales))">
                                        <template v-slot:prepend>
                                        <v-avatar color="blue-grey-lighten-1">
                                            <v-icon>mdi-scale-balance</v-icon>
                                        </v-avatar>
                                        </template>
                                    </v-card>
                                    </v-col>
                                   <br>
                                   <v-col cols="12" md="12" class="pa-1">
                                    <v-card class="mt-4" elevation="2">
                                <v-card-title class="bg-blue-grey-lighten-5">
                                    <v-icon start>mdi-credit-card-multiple</v-icon>
                                    Totales por Método de Pago
                                </v-card-title>
                                <v-data-table
                                    :headers="headersMetodos"
                                    :items="response.totalesPorMetodo || []"
                                    :items-per-page="5"
                                    class="elevation-0"
                                    density="comfortable"
                                    no-data-text="No se encontraron registros de pagos"
                                >
                                    <template v-slot:item.metodo="{ item }">
                                        <v-chip :color="getMethodColor(item.metodo)" size="small" label>
                                        {{ item.metodo }}
                                        </v-chip>
                                    </template>
                                    <template v-slot:item.cantidad="{ item }">
                                        <v-chip variant="outlined" size="small" :color="getMethodColor(item.metodo)">
                                        {{ item.cantidad }}
                                        </v-chip>
                                    </template>
                                    <template v-slot:item.total="{ item }">
                                        <span class="font-weight-bold" :class="'text-' + getMethodColor(item.metodo) + '-darken-3'">
                                        ${{ formatNumber(Number(item.total)) }}
                                        </span>
                                    </template>
                                    <template v-slot:bottom>
                                    <div class="text-right pa-2">
                                        <span class="text-subtitle-1">Total general: </span>
                                        <span class="text-h6 text-success">
                                        ${{ formatNumber(Number(response.totales)) }}
                                        </span>
                                    </div>
                                    </template>
                                </v-data-table>
                                </v-card>
                                </v-col>
                    </v-row>

                    <!-- Separador -->
                    <v-divider class="my-4"></v-divider>

                    <!-- Tramos -->
                    <v-row>
                      <v-col cols="12" class="mt-6">
                    <v-card elevation="1" rounded="lg" class="tramos-card">
                    <v-card-title class="bg-blue-grey-lighten-5 d-flex align-center">
                        <v-icon start>mdi-map-marker-path</v-icon>
                        TRAMOS

                    </v-card-title>
                    
                    <v-card-text class="pa-0 tramos-container">
                        <v-expansion-panels variant="accordion" class="px-2">
                        <v-expansion-panel
                            v-for="(tramo, index) in response.tramos"
                            :key="index"
                            class="my-1"
                        >
                            <v-expansion-panel-title class="py-2" expand-icon="mdi-chevron-down">
                            <v-row align="center" no-gutters>
                                <v-col cols="7" md="8" class="d-flex align-center">
                                <v-avatar :color="tramo.totalPasajes > 0 ? 'green-lighten-4' : 'grey-lighten-3'" size="28" class="mr-2">
                                    <v-icon :color="tramo.totalPasajes > 0 ? 'green-darken-2' : 'grey'" size="16">
                                    mdi-road
                                    </v-icon>
                                </v-avatar>
                                <span class="font-weight-medium text-body-1">{{ tramo.nombre }}</span>
                                </v-col>
                                <v-col cols="5" md="4" class="text-right">
                                <v-tooltip location="top">
                                    <template v-slot:activator="{ props }">
                                    <v-chip
                                        v-bind="props"
                                        :color="tramo.totalPasajes > 0 ? 'green-lighten-2' : 'grey-lighten-2'"
                                        :text-color="tramo.totalPasajes > 0 ? 'green-darken-2' : 'grey-darken-2'"
                                        variant="outlined"
                                        size="medium"
                                        class="mr-1 px-2" 
                                        style="font-size: 1.em;"
                                    >
                                        <v-icon start size="x-small">mdi-ticket</v-icon>
                                        <span class="font-weight-bold">{{ tramo.totalPasajes }}</span>
                                    </v-chip>
                                    </template>
                                    <span>Total pasajes vendidos</span>
                                </v-tooltip>
                                
                                <v-tooltip location="top">
                                    <template v-slot:activator="{ props }">
                                    <v-chip
                                        v-bind="props"
                                        :color="tramo.totalTramo > 0 ? 'blue-lighten-2' : 'grey-lighten-3'"
                                        :text-color="tramo.totalTramo > 0 ? 'blue-darken-2' : 'grey-darken-2'"
                                        size="medium"
                                        class="mr-1 px-2" 
                                        style="font-size: 1.em;"
                                    >
                                        <v-icon start size="x-small">mdi-cash</v-icon>
                                        <span class="font-weight-bold">${{ formatNumber(Number(tramo.totalTramo)) }}</span>
                                    </v-chip>
                                    </template>
                                    <span>Total recaudado</span>
                                </v-tooltip>
                                </v-col>
                            </v-row>
                            </v-expansion-panel-title>
                            
                            <v-expansion-panel-text class="pt-2 pb-3">
                            <v-card variant="flat" class="border">
                                <v-data-table
                                :headers="headersTramoMetodos"
                                :items="tramo.totalesPorMetodo || []"
                                :items-per-page="3"
                                density="compact"
                                class="elevation-0 metodo-pago-table"
                                >
                                <template v-slot:item.metodo="{ item }">
                                    <v-chip :color="getMethodColor(item.metodo)" size="small" label>
                                    {{ item.metodo }}
                                    </v-chip>
                                </template>
                                <template v-slot:item.cantidad="{ item }">
                                    <v-chip variant="outlined" size="small" :color="getMethodColor(item.metodo)">
                                    {{ item.cantidad }}
                                    </v-chip>
                                </template>
                                <template v-slot:item.total="{ item }">
                                    <span class="font-weight-bold" :class="'text-' + getMethodColor(item.metodo) + '-darken-3'">
                                    ${{ formatNumber(Number(item.total)) }}
                                    </span>
                                </template>
                                <template v-slot:bottom>
                                    <div class="text-right pa-2" :class="'bg-' + getMethodColor(tramo.totalesPorMetodo[0]?.metodo) + '-lighten-5'">
                                    <span class="text-caption mr-2">Subtotal:</span>
                                    <span class="text-body-1 font-weight-bold text-success">
                                        ${{ formatNumber(Number(tramo.totalTramo)) }}
                                    </span>
                                    </div>
                                </template>
                                </v-data-table>
                            </v-card>
                            </v-expansion-panel-text>
                        </v-expansion-panel>
                        </v-expansion-panels>
                    </v-card-text>
                    
                    <v-divider v-if="response.tramos?.length > 0"></v-divider>
                    
                    <v-card-actions v-if="response.tramos?.length > 0" class="bg-blue-grey-lighten-5">
                        <v-spacer></v-spacer>
                        <div class="text-right">
                        <span class="text-subtitle-1 mr-2">TOTAL GENERAL:</span>
                        <span class="text-h5 text-green-darken-2 font-weight-bold">
                            ${{ formatNumber(Number(response.tramos.reduce((acc, curr) => acc + Number(curr.totalTramo), 0))) }}
                        </span>
                        </div>
                    </v-card-actions>
                    </v-card>
                    </v-col>
                    </v-row>
                    </v-card-text>
    </v-card>
  </v-container>
</template>

<script>
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import { paleteColors } from "@/assets/colors";
import { format } from "date-fns";
import * as XLSX from "xlsx";
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
    mostrarFila: false,
    dialog: false,
    type: "Sucursal",
    branch_id: "",
    company_id: "",
    role: "",
    response: [],
    branches: [],
    headersResumen: [
      { title: "RESUMEN", key: "title", align: "start", class: "font-weight-bold" },
      { title: "Pasajes emitidos", key: "pasajesEmitidos", align: "end" },
      { title: "Reimpresiones", key: "reimpresiones", align: "end" },
    ],
    headersMetodos: [
      { title: "MÉTODO DE PAGO", key: "metodo", align: "start" },
      { title: "CANTIDAD", key: "cantidad", align: "end" },
      { title: "TOTAL", key: "total", align: "end", class: "font-weight-bold" },
    ],
    headersTramoMetodos: [
      { title: "MÉTODO", key: "metodo", align: "start" },
      { title: "CANTIDAD", key: "cantidad", align: "end" },
      { title: "TOTAL", key: "total", align: "end", class: "font-weight-bold" },
    ],
    data: {},
    menu: false,
    menu2: false,
    input: null,
    input2: null,
    date: null,
    endDate: null,
    options: [
      { title: "Negocio", value: "Company", icon: "mdi-office-building" }, // Opción Negocio con ícono
      { title: "Sucursal", value: "Sucursal", icon: "mdi-store" }, // Opción Sucursal con ícono
    ],
  }),
  computed: {
    dateFormatted() {
      const date = this.input ? new Date(this.input) : new Date();
      const day = date.getDate().toString().padStart(2, "0");
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const year = date.getFullYear();
      return `${year}-${month}-${day}`;
    },
    dateFormatted1() {
      const date = this.input2 ? new Date(this.input2) : new Date();
      const day = date.getDate().toString().padStart(2, "0");
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const year = date.getFullYear();
      return `${year}-${month}-${day}`;
    },
    getDate() {
      return this.input ? new Date(this.input) : new Date();
    },
    getDate2() {
      return this.input2 ? new Date(this.input2) : new Date();
    },
  },
  mounted() {
    this.role = JSON.parse(LocalStorageService.getItem("role"));
    this.company_id = LocalStorageService.getItem("business_id");
    if (this.role === "Administrador") {
      this.showBranches();
      this.type = "Company";
      this.mostrarFila = true;
    } else {
      this.type = "Sucursal";
      this.branch_id = LocalStorageService.getItem("branch_id");
    }
  },
  methods: {
    getMethodColor(metodo) {
    if (!metodo) return 'grey'; // Manejo de valores nulos/undefined
    
    // Normalización del texto
    const normalized = metodo.toString()
        .toLowerCase() // Convertir a minúsculas
        .normalize("NFD").replace(/[\u0300-\u036f]/g, "") // Eliminar tildes
        .trim(); // Eliminar espacios extras
    
    const methodColors = {
        'efectivo': 'green',
        'debito': 'blue',
        'credito': 'orange',
        // Puedes agregar más variantes si es necesario
        'tarjeta debito': 'blue',
        'tarjeta credito': 'orange',
        'cash': 'green',
        'contado': 'green'
    };
    
    return methodColors[normalized] || 'grey';
    },
    async showBranches() {
      try {
        const result = await handleRequest({
          endpoint: "branch",
          method: "GET",
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.branches = result.data?.branches || [];
          this.branch_id = this.branches[0].id;
        } else {
          // Si no hay datos, asignamos un array vacío
          this.branches = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        //this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
      } finally {
        this.loading = false;
        this.initialize();
      }
    },
    async initialize() {
      this.response = [];
      try {
        this.loading = true;
        this.data = {};
        this.data.id =
          this.type === "Company" ? Number(this.company_id) : Number(this.branch_id);
        this.data.type = this.type;
        // Formatear las fechas
        const formattedDate = this.date
          ? format(new Date(this.date), "yyyy-MM-dd")
          : format(new Date(), "yyyy-MM-dd");
        const formattedEndDate = this.endDate
          ? format(new Date(this.endDate), "yyyy-MM-dd")
          : format(new Date(), "yyyy-MM-dd");

        // Comparar las fechas
        if (formattedDate === formattedEndDate) {
          this.data.date = formattedDate; // Solo enviar una fecha si son iguales
        } else {
          this.data.date = formattedDate;
          this.data.endDate = formattedEndDate;
        }
        const result = await handleRequest({
          endpoint: "trips-tickets-date",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.response = result.data || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.response = [];
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
    formatNumber(value) {
      // Si el valor es menor que 1000, devuelve el valor original con dos decimales
      if (value < 1000) {
        return (Math.round((value + Number.EPSILON) * 100) / 100).toLocaleString(
          "en-US",
          { minimumFractionDigits: 2, maximumFractionDigits: 2 }
        );
      }

      // Primero, redondea el valor a dos decimales
      value = Math.round((value + Number.EPSILON) * 100) / 100;

      // Convierte el valor a cadena con formato de número local (en-US)
      let formattedValue = value.toLocaleString("en-US", {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      });

      return formattedValue;
    },
    updateDate(val) {
      this.input = val;
      this.date = this.dateFormatted;
      this.menu = false;
    },
    updateDate1(val) {
      this.input2 = val;
      this.endDate = this.dateFormatted1;
      this.menu2 = false;
    },
    exportToExcel() {
      let rows = [];

      // 1. Título principal
      rows.push([this.response.nombre]);
      rows.push([]); // Fila vacía para separar

      // 2. Fecha
      rows.push(["FECHA:", this.response.fecha]);
      rows.push([]); // Fila vacía para separar

      // 3. Resumen
      rows.push(["RESUMEN:"]);
      rows.push([]); // Fila vacía para separar

      // 4. Emisión de pasajes
      rows.push(["EMISIÓN DE PASAJES:"]);
      rows.push([]); // Fila vacía para separar

      // 5. Pasajes emitidos y reimpresiones
      rows.push(["Pasajes emitidos:", this.response.pasajesEmitidos]);
      rows.push(["Reimpresiones:", this.response.reimpresiones]);
      rows.push([]); // Fila vacía para separar

      // 6. Totales por método de pago (cantidad)
      this.response.totalesPorMetodo.forEach((item) => {
        rows.push([`${item.metodo}:`, item.cantidad]);
      });
      rows.push([]); // Fila vacía para separar

      // 7. Encabezado de totales
      rows.push(["TOTALES:"]);
      rows.push([]); // Fila vacía para separar

      // 8. Totales por método de pago (monto)
      this.response.totalesPorMetodo.forEach((item) => {
        rows.push([`${item.metodo}:`, `$${item.total}`]);
      });
      rows.push([]); // Fila vacía para separar

      // 9. Total general
      rows.push(["TOTAL:", `$${this.response.totales}`]);
      rows.push([]); // Fila vacía para separar

      // 10. Sección de tramos
      rows.push(["---------------------------------------------"]);
      rows.push(["TRAMOS:"]);
      rows.push(["---------------------------------------------"]);
      rows.push([]); // Fila vacía para separar

      // 11. Datos de cada tramo
      this.response.tramos.forEach((tramo) => {
        rows.push([tramo.nombre]); // Nombre del tramo
        rows.push([]); // Fila vacía para separar

        // Total de pasajes del tramo
        rows.push(["Total Pasajes:", tramo.totalPasajes]);

        // Totales por método de pago (cantidad) en el tramo
        tramo.totalesPorMetodo.forEach((total) => {
          rows.push([`${total.metodo}:`, total.cantidad]);
        });

        // Total del tramo
        rows.push(["Total Tramo:", tramo.totalTramo]);

        // Totales por método de pago (monto) en el tramo
        tramo.totalesPorMetodo.forEach((total) => {
          rows.push([`${total.metodo}:`, `$${total.total}`]);
        });

        // Separador entre tramos
        rows.push(["---------------------------------------------"]);
        rows.push([]); // Fila vacía para separar
      });

      // 12. Crear el archivo Excel
      const ws = XLSX.utils.aoa_to_sheet(rows);
      const wb = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(wb, ws, "Reporte");

      // 13. Guardar el archivo
      XLSX.writeFile(
        wb,
        `reporte_ventas_${new Date().toLocaleDateString().replace(/\//g, "-")}.xlsx`
      );
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
    exportToExcel1() {
  let rows = [];

  // 1. Encabezado
  rows.push([this.response.nombre]);
  rows.push(["FECHA:", this.response.fecha]);
  rows.push([]);

  // 2. Resumen
  rows.push(["RESUMEN"]);
  rows.push(["Pasajes emitidos:", this.response.pasajesEmitidos]);
  rows.push(["Reimpresiones:", this.response.reimpresiones]);
  rows.push([]);

  // 3. Métodos de pago
  rows.push(["TOTALES POR MÉTODO DE PAGO"]);
  this.response.totalesPorMetodo.forEach(item => {
    rows.push([item.metodo, item.cantidad, `$${item.total}`]);
  });
  rows.push([]);

  // 4. Total general
  rows.push(["TOTAL GENERAL", `$${this.response.totales}`]);
  rows.push([]);

  // 5. Tramos
  rows.push(["TRAMOS"]);
  rows.push([]);
  
  this.response.tramos.forEach(tramo => {
    rows.push([tramo.nombre]);
    rows.push(["Total Pasajes:", tramo.totalPasajes]);
    rows.push(["Total Tramo:", `$${tramo.totalTramo}`]);
    
    if (tramo.totalesPorMetodo.length > 0) {
      rows.push(["Métodos de pago:"]);
      tramo.totalesPorMetodo.forEach(item => {
        rows.push([item.metodo, item.cantidad, `$${item.total}`]);
      });
    } else {
      rows.push(["No hay ventas registradas"]);
    }
    rows.push([]);
  });

  // Crear el archivo Excel
  const ws = XLSX.utils.aoa_to_sheet(rows);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Reporte");

  XLSX.writeFile(wb, `reporte_ventas_${new Date().toLocaleDateString().replace(/\//g, '-')}.xlsx`);
}
  },
};
</script>

<style scoped>
/* Estilos personalizados */

.v-card-title {
  padding: 16px;
}

.v-card-text {
  padding: 16px;
}

.v-table {
  width: 100%;
}

.text-h4 {
  font-weight: bold;
}

.primary--text {
  color: #1976d2;
  /* Color primario de Vuetify */
}

.tramos-card {
  max-height: 70vh; /* Altura máxima ajustable */
  display: flex;
  flex-direction: column;
}

.tramos-container {
  overflow-y: auto;
  flex: 1;
}

.metodo-pago-table {
  max-height: 200px;
}

/* Scrollbar personalizada */
.tramos-container::-webkit-scrollbar {
  width: 6px;
}

.tramos-container::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.tramos-container::-webkit-scrollbar-thumb {
  background: #b0bec5;
  border-radius: 3px;
}

.tramos-container::-webkit-scrollbar-thumb:hover {
  background: #78909c;
}

</style>
