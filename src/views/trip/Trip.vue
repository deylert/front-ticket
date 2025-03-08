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
  <v-container style="min-width: 100%;">
    <v-card elevation="6" class="mx-2">
      <v-toolbar :color="paleteColors.primary">
        <v-row align="center">
          <v-col cols="12" md="8" class="grow ml-4">
            <span class="text-subtitle-1"><strong>Viajes</strong></span>
          </v-col>
          <v-col cols="12" md="3" class="text-right">
            <v-btn class="text-subtitle-1 ml-12" :color="paleteColors.white" variant="tonal" elevation="2"
              prepend-icon="mdi-plus-circle" @click="showAdd">
              Agregar Viaje
            </v-btn>
          </v-col>
        </v-row>
      </v-toolbar>
      <v-card-text>
        <v-row>
          <v-container fluid>
            <v-cols cols="12" md="12">
              <v-row v-if="mostrarFila" dense>
                <v-col cols="12" md="3">
                  <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="branch_id" :items="branches"
                    label="Seleccione una Sucursal" prepend-inner-icon="mdi-store" item-title="name" item-value="id"
                    variant="underlined" :rules="selectRules" density="compact">
                    <template v-slot:item="{ props, item }">
                      <v-list-item v-bind="props"
                        :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`">
                      </v-list-item>
                    </template>
                  </v-autocomplete><!-- @update:model-value="initialize()">-->
                </v-col>
                <v-col cols="12" md="2">
                  <v-btn icon @click="initialize" :color="paleteColors.primary">
                    <v-icon>mdi-magnify</v-icon></v-btn>
                </v-col>
              </v-row>
            </v-cols>
          </v-container>
        </v-row>
        <v-row dense>
          <v-col cols="12">
            <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
              hide-details>
            </v-text-field>
            <v-data-table :headers="headers" :search="search" :items="trips" class="elevation-1"
              style="max-height: 68vh; overflow-y: auto" :items-per-page-text="'Elementos por páginas'"
              no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
              <template v-slot:item.actions="{ item }">
                <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)" :color="paleteColors.primary" variant="tonal"
                  elevation="1" title="Editar Viaje"></v-btn>
                <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" :color="paleteColors.error" variant="tonal"
                  elevation="1" title="Eliminar Viaje"></v-btn>
              </template>
              <template v-slot:item.origin="{ item }">
                <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                  <v-img :src="`${this.$axios.defaults.baseURL}images/${item.originImage
                    }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
                {{ item.origin }}
              </template>
              <template v-slot:item.destination="{ item }">
                <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                  <v-img :src="`${this.$axios.defaults.baseURL}images/${item.destinationImage
                    }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
                {{ item.destination }}
              </template>
              <template v-slot:item.vehicleName="{ item }">
                <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                  <v-img :src="`${this.$axios.defaults.baseURL}images/${item.vehicleImage
                    }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
                {{ item.vehicleName }}
              </template>
            </v-data-table>
          </v-col>
        </v-row>

      </v-card-text>
    </v-card>
  </v-container>

  <v-dialog v-model="dialog" max-width="700px">
    <v-form ref="form" v-model="valid" enctype="multipart/form-data">
      <v-card>
        <v-toolbar :color="paleteColors.primary">
          <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
        </v-toolbar>
        <v-card-text>
          <v-container>
            <v-tabs v-model="tab" vertical>
              <!-- Pestañas -->
              <v-tab value="general" :class="tab === 'general' ? 'selected-tab' : ''">Generales</v-tab>
              <v-tab value="worker" :class="tab === 'worker' ? 'selected-tab' : ''"
                v-if="editedItem.vehicle_id">Trabajadores</v-tab>
            </v-tabs>
            <!-- Contenido de las pestañas -->
            <v-window v-model="tab">
              <v-window-item value="general">
                <v-row style="margin-top: 5px">
                  <v-col cols="12" md="12">
                    <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.route_id"
                      :items="routes" label="Ruta" prepend-icon="mdi-road" item-title="name" item-value="id"
                      variant="underlined" :rules="selectRules" density="compact" @update:model-value="updateStimated">
                      <template v-slot:item="{ props, item }">
                        <v-list-item v-bind="props">
                          <v-list-item-content>
                            <!-- Subtítulo con los avatares e información de origen y destino -->
                            <v-list-item-subtitle>
                              <v-row align="center" no-gutters>
                                <!-- Origen -->
                                <v-col cols="auto" class="d-flex align-center">
                                  <v-avatar>
                                    <v-img :src="`${this.$axios.defaults.baseURL}images/${item.raw.originImage}`"
                                      max-width="40" />
                                  </v-avatar>
                                  <div class="ml-2" style="
                                      max-width: 150px;
                                      white-space: nowrap;
                                      overflow: hidden;
                                      text-overflow: ellipsis;
                                    ">
                                    {{ item.raw.originAddress }}
                                  </div>
                                </v-col>

                                <!-- Destino -->
                                <v-col cols="auto" class="d-flex align-center">
                                  <v-avatar>
                                    <v-img :src="`${this.$axios.defaults.baseURL}images/${item.raw.destinationImage}`"
                                      max-width="40" />
                                  </v-avatar>
                                  <div class="ml-2" style="
                                      max-width: 150px;
                                      white-space: nowrap;
                                      overflow: hidden;
                                      text-overflow: ellipsis;
                                    ">
                                    {{ item.raw.destinationAddress }}
                                  </div>
                                </v-col>
                              </v-row>
                            </v-list-item-subtitle>
                          </v-list-item-content>
                        </v-list-item>
                      </template>
                    </v-autocomplete>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.vehicle_id"
                      :items="vehicles" label="Vehículo" prepend-icon="mdi-car-side" item-title="vehicleName"
                      item-value="id" variant="underlined" :rules="selectRules" density="compact"
                      @update:model-value="filterWorkers">
                      <template v-slot:item="{ props, item }">
                        <v-list-item v-bind="props"
                          :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.vehicleImage}`"
                          :title="item.raw.vehicleName">
                          <v-list-item-subtitle class="d-flex flex-column">
                            <div>Marca: {{ item.raw.brand }}</div>
                            <div>Asientos: {{ item.raw.seats }}</div>
                          </v-list-item-subtitle>
                        </v-list-item>
                      </template>
                    </v-autocomplete>
                  </v-col>
                  <v-col cols="12" md="6">
                    <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
                      transition="scale-transition" offset-y min-width="290px">
                      <template v-slot:activator="{ props }">
                        <v-text-field v-bind="props" :modelValue="dateFormatted" variant="underlined"
                          prepend-icon="mdi-calendar" label="Fecha" density="compact"></v-text-field>
                      </template>
                      <v-locale-provider locale="es">
                        <v-date-picker header="Calendario" title="Seleccione la fecha" :color="paleteColors.primary"
                          :modelValue="input" @update:model-value="updateDate" format="yyyy-MM-dd"
                          :min="new Date().toISOString().split('T')[0]"></v-date-picker>
                      </v-locale-provider>
                    </v-menu>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" md="4">
                    <v-select v-model="editedItem.schedule" :items="timeSlots" label="Hora de salida"
                      variant="underlined" density="compact" clearable prepend-icon="mdi-calendar-clock"
                      @update:model-value="updateArrival"></v-select>
                  </v-col>
                  <v-col cols="12" md="4">
                    <v-select v-model="editedItem.arrival" :items="timeSlots" label="Hora de llegada"
                      variant="underlined" density="compact" clearable prepend-icon="mdi-calendar-clock"></v-select>
                  </v-col>
                  <v-col cols="12" md="4">
                    <v-text-field v-model="editedItem.price" clearable label="Precio" prepend-icon="mdi-currency-usd"
                      variant="underlined" :rules="priceRules" type="number" density="compact"></v-text-field>
                  </v-col>
                </v-row>
              </v-window-item>
              <v-window-item value="worker" class="mt-4">
                <v-card elevation="3" class="mx-2">
                  <v-toolbar :color="paleteColors.primary">
                    <v-row align="center">
                      <v-col cols="12" md="7" class="grow ml-4">
                        <span class="text-subtitle-1"><strong>Relación de Trabajadores</strong></span>
                      </v-col>
                      <v-col cols="12" md="4" class="text-right">
                        <v-btn class="text-subtitle-1" :color="paleteColors.white" variant="tonal" elevation="2"
                          prepend-icon="mdi-plus-circle" @click="showAssiegnedWorker">
                          Asignar Trabajador
                        </v-btn>
                      </v-col>
                    </v-row>
                  </v-toolbar>

                  <v-card-text>
                    <v-data-table :headers="headersWorkers" :items="editedItem.workers" class="elevation-1"
                      style="max-height: 68vh; overflow-y: auto" :items-per-page-text="'Elementos por páginas'"
                      no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
                      <template v-slot:item.actions="{ item }">
                        <v-btn density="comfortable" icon="mdi-delete" @click="deleteItemWorker(item)" :color="paleteColors.error"
                          variant="tonal" elevation="1" title="Eliminar Relación"></v-btn>
                      </template>
                      <template v-slot:item.name="{ item }">
                        <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="large">
                          <v-img :src="`${this.$axios.defaults.baseURL}images/${item.image
                            }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
                        {{ item.name }}
                      </template>
                    </v-data-table>
                  </v-card-text>
                </v-card>
              </v-window-item>
            </v-window>
          </v-container>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn :color="paleteColors.gris" variant="flat" @click="close">Cancelar</v-btn>
          <v-btn :color="paleteColors.primary" variant="flat" @click="save" :disabled="(!valid || !editedItem.workers.length)"
            :loading=loading>Aceptar</v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
  <v-dialog v-model="dialogDelete" max-width="500px">
    <v-card>
      <v-toolbar :color="paleteColors.error">
        <span class="text-subtitle-2 ml-4"> Eliminar un viaje</span>
      </v-toolbar>

      <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el viaje seleccionado?</v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn :color="paleteColors.gris" variant="flat" @click="closeDelete"> Cancelar </v-btn>
        <v-btn :color="paleteColors.error" variant="flat" @click="deleteItemConfirm"> Aceptar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>

  <v-dialog v-model="dialogAssignedWorkers" max-width="400px">
    <v-form ref="form" v-model="valid" enctype="multipart/form-data">
      <v-card>
        <v-toolbar :color="paleteColors.primary">
          <span class="text-subtitle-2 ml-4">Asignar trabajadores al viaje</span>
        </v-toolbar>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" md="12">
                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="selectedWorker"
                  :items="filteredWorkers" label="Personas" prepend-icon="mdi-account" item-title="workerName"
                  item-value="id" variant="underlined" :rules="selectRules">
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props"
                      :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.workerImage}`"
                      :title="item.raw.workerName">
                      <v-list-item-subtitle class="d-flex flex-column">
                        <div>Rol: {{ item.raw.roleName }}</div>
                      </v-list-item-subtitle>
                    </v-list-item>
                  </template>
                </v-autocomplete>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn :color="paleteColors.gris" variant="flat" @click="closeAssignedWorker">Cancelar</v-btn>
          <v-btn :color="paleteColors.primary" variant="flat" @click="saveAssignedWorker" :disabled="!valid">Aceptar</v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
</template>

<script>
import { paleteColors } from "@/assets/colors";
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import _ from 'lodash';
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
    estimated: null,
    mostrarFila: false,
    trips: [],
    routes: [],
    vehicles: [],
    workers: [],
    branches: [],
    filteredWorkers: [],
    data: {},
    selectedWorker: "",
    route: '',
    branch_id: '',
    dialogAssignedWorkers: false,
    headers: [
      { title: "Ruta", value: "name", width: "15%" },
      { title: "Origen", value: "origin", width: "20%" },
      { title: "Destino", value: "destination", width: "20%" },
      { title: "Vehículo", value: "vehicleName", width: "15%" },
      { title: "Fecha", value: "date", width: "4%" },
      { title: "Horario", value: "schedule", width: "4%" },
      { title: "Precio", value: "price", width: "4%" },
      { title: "Salida", value: "start", width: "4%" },
      { title: "Llegada", value: "end", width: "4%" },
      { title: "Acciones", value: "actions", sortable: false, width: "10%" },
    ],

    headersWorkers: [
      { title: "Nombre", value: "name", width: "60%" },
      { title: "Rol", value: "roleName", width: "20%" },
      { title: "Acciones", value: "actions", sortable: false, width: "20%" },
    ],

    editedItem: {
      id: "",
      route_id: "",
      branch_id: "",
      vehicle_id: "",
      date: "",
      schedule: "",
      arrival: "",
      start: "",
      end: "",
      price: "",
      workers: [],
    },
    originalItem: {
      id: "",
      route_id: "",
      branch_id: "",
      vehicle_id: "",
      date: "",
      schedule: "",
      arrival: "",
      start: "",
      end: "",
      price: "",
      workers: [],
    },
    defaultItem: {
      id: "",
      route_id: "",
      branch_id: "",
      vehicle_id: "",
      date: "",
      schedule: "",
      arrival: "",
      start: "",
      end: "",
      price: "",
      workers: [],
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
    formTitle() {
      return this.editedIndex === -1 ? "Agregar Viaje" : "Editar Viaje";
    },
    dateFormatted() {
      const date = this.input ? new Date(this.input) : new Date();
      const day = date.getDate().toString().padStart(2, "0");
      const month = (date.getMonth() + 1).toString().padStart(2, "0");
      const year = date.getFullYear();
      return `${year}-${month}-${day}`;
    },
    getDate() {
      return this.input ? new Date(this.input) : new Date();
    },
  },
  mounted() {
    this.role = JSON.parse(LocalStorageService.getItem('role'));
    if (this.role === 'Administrador') {
      this.showBranches();
    } else {
      this.branch_id = LocalStorageService.getItem('branch_id');
    }
    this.timeSlots = this.generateTimeSlots();
  },
  methods: {
    async showBranches() {
      try {
        const result = await handleRequest({
          endpoint: 'branch',
          method: 'GET',
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
        this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
      } finally {
        this.mostrarFila = true;
        this.loading = false;
        this.initialize();
      }
    },
    // Filtramos los trabajadores según el vehículo seleccionado
    filterWorkers() {
      const selectedVehicleId = this.editedItem.vehicle_id;
      // Filtramos los trabajadores que están relacionados con el vehículo seleccionado
      this.filteredWorkers = this.workers.filter((worker) =>
        worker.vehicles.some((vehicle) => vehicle.id === selectedVehicleId)
      );
    },
    updateStimated() {
      this.estimated = null;
      const matchedRoute = this.routes.find((route) => route.id === this.editedItem.route_id);
      // Si se encuentra el objeto, asignamos su propiedad 'estimated' a this.estimated
      this.estimated = matchedRoute ? matchedRoute.estimated : null;
    },
    updateArrival() {
      if (!this.editedItem.schedule || !this.estimated) {
        this.editedItem.arrival = null; // Manejo de casos donde no hay valores válidos
        return;
      }

      // Convertir el valor de schedule (hora:minuto) en minutos totales
      const [hours, minutes] = this.editedItem.schedule.split(":").map(Number);
      const scheduleInMinutes = hours * 60 + minutes;

      // Sumar el tiempo estimado (this.estimated)
      const arrivalInMinutes = scheduleInMinutes + this.estimated;

      // Convertir minutos totales de llegada a formato hora:minuto
      const arrivalHours = Math.floor(arrivalInMinutes / 60) % 24; // Aseguramos que sea un formato de 24 horas
      const arrivalMinutes = arrivalInMinutes % 60;

      const formattedArrival = `${String(arrivalHours).padStart(2, "0")}:${String(arrivalMinutes).padStart(2, "0")}`;

      // Asignar el valor calculado a editedItem.arrival
      this.editedItem.arrival = formattedArrival;
    },
    generateTimeSlots() {
      const slots = [];
      for (let hour = 0; hour < 24; hour++) {
        for (let minute = 0; minute < 60; minute += 5) {
          // Incrementos de 5 minutos
          const formattedHour = String(hour).padStart(2, "0");
          const formattedMinute = String(minute).padStart(2, "0");
          slots.push(`${formattedHour}:${formattedMinute}`);
        }
      }
      return slots;
    },
    updateDate(val) {
      this.input = val;
      this.editedItem.date = this.dateFormatted;
      this.menu = false;
    },
    async showAdd() {
      this.data = {};
      this.filteredWorkers = [];
      this.data.branch_id = this.branch_id;
      this.editedIndex = -1;
      this.editedItem = Object.assign({}, this.defaultItem);
      this.originalItem = Object.assign({}, this.defaultItem);
      try {
        const result = await handleRequest({
          endpoint: "get-routes-vehicle-workers",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.routes = result.data?.triproutes || [];
          this.vehicles = result.data?.tripvehicles || [];
          this.workers = result.data.tripworkers || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.routes = [];
          this.vehicles = [];
          this.workers = [];
        }
      } catch (error) {
        this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
      } finally {
        this.dialog = true;
      }
    },
    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.originalItem = Object.assign({}, this.defaultItem);
      });
      this.file = null;
      this.imgMiniatura = "";
      this.editedIndex = -1;
    },
    async showAssiegnedWorker() {
      // Clonar filteredWorkers para evitar referencias compartidas
      const clonedFilteredWorkers = this.filteredWorkers.map(worker => ({ ...worker }));

      // Ahora puedes filtrar el arreglo clonado sin afectar a los objetos originales
      this.filteredWorkers = clonedFilteredWorkers.filter(worker => {
        // Verificar si la persona no está en editedItem.workers
        return !this.editedItem.workers.some(editedWorker => editedWorker.id === worker.id);
      });
      this.dialogAssignedWorkers = true;
    },
    closeAssignedWorker() {
      this.dialogAssignedWorkers = false;
      this.selectedWorker = null;
    },
    saveAssignedWorker() {
      if (this.selectedWorker) {
        const worker = this.workers.find((p) => p.id === this.selectedWorker);
        const newWorkers = {
          id: worker.id,
          name: worker.workerName,
          image: worker.workerImage,
          roleId: worker.roleId,
          roleName: worker.roleName,
        };
        // Verificar si la relación ya existe en editedItem.people
        const existingPersonIndex = this.editedItem.workers.findIndex(p =>
          p.id === newWorkers.id);

        if (existingPersonIndex === -1) {
          // No existe, por lo tanto, se agrega uno nuevo
          this.editedItem.workers.push(newWorkers);
        } else {
          // Existe, por lo tanto se edita el existente
          this.editedItem.workers.splice(existingPersonIndex, 1, newWorkers); // Actualiza el elemento en el array
        }
      }

      // Reiniciar selección y cerrar diálogo
      this.closeAssignedWorker();
    },
    deleteItemWorker(item) {
      const index = this.editedItem.workers.findIndex((p) => p.id === item.id);
      if (index !== -1) {
        this.editedItem.workers.splice(index, 1);
      }
    },
    areWorkersDifferent(originalWorkers, editedWorkers) {
      if (originalWorkers.length !== editedWorkers.length) {
        return true; // Si tienen longitudes diferentes, son diferentes
      }

      // Ordenar ambos arrays para una comparación consistente
      const sortedOriginal = [...originalWorkers].sort((a, b) => a.id - b.id);
      const sortedEdited = [...editedWorkers].sort((a, b) => a.id - b.id);

      // Comparar cada objeto en los arrays
      return sortedOriginal.some((original, index) => {
        const edited = sortedEdited[index];
        return Object.keys(original).some(
          (key) => original[key] !== edited[key]
        );
      });
    },
    async initialize() {
      this.data = {};
      this.data.branch_id = this.branch_id;
      const today = new Date();
      const formattedDate = today.toISOString().split('T')[0]; // Formato: YYYY-MM-DD
      //this.data.date = formattedDate;
      try {
        this.loading = true;
        const result = await handleRequest({
          endpoint: "get-trip-branch-date",
          method: "POST",
          data: this.data
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.trips = result.data?.trips || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.trips = [];
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
    async save() {
      this.loading = true;
      if (this.editedIndex === -1) {
        this.valid = false;
        const fieldsToUpdate = [
          "route_id",
          "branch_id",
          "vehicle_id",
          "date",
          "schedule",
          "arrival",
          "start",
          "end",
          "price",
          "workers",
        ];

        let updatedFields = Object.keys(this.editedItem)
          .filter(
            (key) =>
              fieldsToUpdate.includes(key) && (this.originalItem[key], this.editedItem[key]))
          .reduce((obj, key) => {
            if (key === "workers") {
              // Transformar el campo `people`
              obj[key] = this.editedItem.workers.map((worker) => ({
                worker_id: Number(worker.id), // Asegurar que sea un número
              }));
            } else {
              obj[key] = this.editedItem[key];
            }
            return obj;
          }, {});
        if (Object.keys(updatedFields).length > 0) {
          updatedFields.date = this.editedItem.date ? this.editedItem.date : new Date();
          updatedFields.branch_id = this.branch_id;
          try {
            const result = await handleRequest({
              endpoint: "trip",
              method: "POST",
              data: updatedFields,
            });

            // Manejo de la respuesta según el resultado
            if (result.success) {
              this.showAlert("success", result.message, 3000);
              this.initialize();
              this.loading = false;
            } else {
              this.showAlert("warning", result.message, 3000);
              this.loading = false;
              this.valid = true;
            }
          } catch (error) {
            // Este bloque captura errores inesperados fuera del manejo estándar
            this.showAlert(
              "error",
              "Ocurrió un error inesperado al procesar la solicitud.",
              3000
            );
            this.loading = false;
            this.valid = true;
          }
        }
      } else {
        this.valid = false;
        const fieldsToUpdate = [
          "route_id",
          "branch_id",
          "vehicle_id",
          "date",
          "schedule",
          "arrival",
          "start",
          "end",
          "price",
          "workers",
        ];
        let updatedFields = Object.keys(this.editedItem)
          .filter(
            (key) =>
              fieldsToUpdate.includes(key) &&
              (key !== "workers"
                ? this.editedItem[key] !== this.originalItem[key]
                : this.areWorkersDifferent(this.originalItem[key], this.editedItem[key])) // Compara el array people
          )
          .reduce((obj, key) => {
            if (key === "workers") {
              // Transformar el campo `people`
              obj[key] = this.editedItem.workers.map((worker) => ({
                worker_id: Number(worker.id), // Asegurar que sea un número
              }));
            } else {
              obj[key] = this.editedItem[key];
            }
            return obj;
          }, {});
        if (Object.keys(updatedFields).length > 0) {
          updatedFields.id = this.editedItem.id;
          try {
            const result = await handleRequest({
              endpoint: "trip",
              method: "PUT",
              data: updatedFields,
            });

            // Manejo de la respuesta según el resultado
            if (result.success) {
              this.showAlert("success", result.message, 3000);
              this.initialize();
              this.loading = false;
            } else {
              this.showAlert("warning", result.message, 3000);
              this.loading = false;
            }
          } catch (error) {
            // Este bloque captura errores inesperados fuera del manejo estándar
            this.showAlert(
              "error",
              "Ocurrió un error inesperado al procesar la solicitud.",
              3000
            );
            this.loading = false;
          }
        } else {
          this.showAlert("success", "No se realizaron cambios.", 3000);
          this.loading = false;
        }
      }
      this.close();
    },
    async editItem(item) {
      this.editedIndex = 1;
      this.originalItem = _.cloneDeep(item);
      this.editedItem = _.cloneDeep(item);
      this.data = {};
      this.data.branch_id = this.branch_id;
      try {
        const result = await handleRequest({
          endpoint: "get-routes-vehicle-workers",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.routes = result.data?.triproutes || [];
          this.vehicles = result.data?.tripvehicles || [];
          this.workers = result.data.tripworkers || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.routes = [];
          this.vehicles = [];
          this.workers = [];
        }
      } catch (error) {
        this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
      } finally {
        this.updateStimated();
        this.filterWorkers();
        this.dialog = true;
      }
    },
    deleteItem(item) {
      this.editedIndex = 1;
      this.editedItem.id = item.id;
      this.dialogDelete = true;
    },
    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
      });
    },
    async deleteItemConfirm() {
      try {
        let request = {
          id: this.editedItem.id,
        };
        const result = await handleRequest({
          endpoint: "trip-destroy",
          method: "POST",
          data: request,
        });

        // Manejo de la respuesta según el resultado
        if (result.success) {
          this.showAlert("success", result.message, 3000);
          this.initialize();
        } else {
          this.showAlert("warning", result.message, 3000);
        }
      } catch (error) {
        // Este bloque captura errores inesperados fuera del manejo estándar
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al procesar la solicitud.",
          3000
        );
      } finally {
        this.closeDelete();
      }
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
.selected-tab {
  background-color: #1976D2;
  /* Fondo del tab seleccionado */
  color: white;
  /* Texto blanco */
  border-radius: 4px;
  /* Esquinas redondeadas, opcional */
}
</style>
