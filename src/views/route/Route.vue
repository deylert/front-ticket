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
      <v-toolbar color="#1976D2">
        <v-row align="center">
          <v-col cols="12" md="8" class="grow ml-4">
            <span class="text-subtitle-1"><strong>Rutas</strong></span>
          </v-col>
          <v-col cols="12" md="3" class="text-right">
            <v-btn class="text-subtitle-1 ml-12" color="white" variant="tonal" elevation="2"
              prepend-icon="mdi-plus-circle" @click="showAdd">
              Agregar Ruta
            </v-btn>
          </v-col>
        </v-row>
      </v-toolbar>

      <v-card-text>
        <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
          hide-details>
        </v-text-field>
        <v-data-table :headers="headers" :search="search" :items="routes" class="elevation-1"
          style="max-height: 68vh; overflow-y: auto" :items-per-page-text="'Elementos por páginas'"
          no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
          <template v-slot:item.actions="{ item }">
            <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)" color="#1976D2" variant="tonal"
              elevation="1" title="Editar Ruta"></v-btn>
            <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" color="#DA7171" variant="tonal"
              elevation="1" title="Eliminar Ruta"></v-btn>
          </template>
          <template v-slot:item.originAddress="{ item }">
            <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
              <v-img :src="`${this.$axios.defaults.baseURL}images/${item.originImage
                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
            {{ item.originAddress }}
          </template>
          <template v-slot:item.destinationAddress="{ item }">
            <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
              <v-img :src="`${this.$axios.defaults.baseURL}images/${item.destinationImage
                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
            {{ item.destinationAddress }}
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </v-container>
  <v-dialog v-model="dialog" max-width="500px">
    <v-form ref="form" v-model="valid" enctype="multipart/form-data">
      <v-card>
        <v-toolbar color="#1976D2">
          <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
        </v-toolbar>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" md="12">
                <v-text-field v-model="editedItem.name" clearable label="Nombre" prepend-icon="mdi-store"
                  variant="underlined" :rules="nameRules"></v-text-field>
              </v-col>
              <v-col cols="12" md="12">
                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.origin_id"
                  :items="locations" label="Origen" prepend-icon="mdi-arrow-up-circle" item-title="address"
                  item-value="id" variant="underlined" :rules="selectRules">
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props"
                      :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`"
                      :title="item.raw.address"></v-list-item>
                  </template>
                </v-autocomplete>
              </v-col>
              <v-col cols="12" md="12">
                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.destination_id"
                  :items="locations" label="Destino" prepend-icon="mdi-arrow-down-circle" item-title="address"
                  item-value="id" variant="underlined" :rules="selectRules">
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props"
                      :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`"
                      :title="item.raw.address"></v-list-item>
                  </template>
                </v-autocomplete>
              </v-col>
              <v-col cols="12" md="12">
                <v-text-field v-model="editedItem.distance" clearable label="Distancia" prepend-icon="mdi-ruler"
                  variant="underlined" :rules="durationRules"></v-text-field>
              </v-col>
              <v-col cols="12" md="12">
                <v-text-field v-model="editedItem.estimated" clearable label="Duración (Minutos)"
                  prepend-icon="mdi-timer" variant="underlined" :rules="durationRules"></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="#DA7171" variant="flat" @click="close">Cancelar</v-btn>
          <v-btn color="#1976D2" variant="flat" @click="save" :disabled="!valid" :loading="loading">Aceptar</v-btn>
        </v-card-actions>
      </v-card>
    </v-form>
  </v-dialog>
  <v-dialog v-model="dialogDelete" max-width="500px">
    <v-card>
      <v-toolbar color="#DA7171">
        <span class="text-subtitle-2 ml-4"> Eliminar una Ruta</span>
      </v-toolbar>

      <v-card-text class="mt-2 mb-2"> ¿Desea eliminar la ruta?</v-card-text>
      <v-divider></v-divider>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="#DA7171" variant="flat" @click="closeDelete"> Cancelar </v-btn>
        <v-btn color="#1976D2" variant="flat" @click="deleteItemConfirm"> Aceptar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { handleRequest } from "@/utils/api"; // Ruta al archivo
export default {
  data: () => ({
    snackbar: false,
    sb_type: "",
    sb_message: "",
    sb_timeout: 2000,
    sb_title: "",
    sb_icon: "",
    valid: true,
    loading: false,
    mostrar: false,
    dialog: false,
    dialogDelete: false,
    locations: [],
    routes: [],
    data: {},
    headers: [
      { title: "Nombre", value: "name", width: "15%" },
      { title: "Origen", value: "originAddress", width: "30%" },
      { title: "Destino", value: "destinationAddress", width: "30%" },
      { title: "Distancia", value: "distance", width: "5%" },
      { title: "Duración", value: "estimated", width: "5%" },
      { title: "Acciones", value: "actions", sortable: false, width: "15%" },
    ],

    editedItem: {
      id: "",
      name: "",
      origin_id: "",
      destination_id: "",
      distance: "",
      estimated: "",
      status: "1",
    },
    originalItem: {
      id: "",
      name: "",
      origin_id: "",
      destination_id: "",
      distance: "",
      estimated: "",
      status: "1",
    },
    defaultItem: {
      id: "",
      name: "",
      origin_id: "",
      destination_id: "",
      distance: "",
      estimated: "",
      status: "1",
    },
    editedIndex: -1,
    search: "",
    nameRules: [
      (v) => !!v || "El campo es requerido",
      (v) => (v && v.length <= 50) || "El campo debe tener menos de 51 caracteres",
      (v) => (v && v.length >= 3) || "El campo debe tener al menos 3 caracteres",
    ],
    selectRules: [(v) => !!v || "Seleccionar al menos un elemento"],
    distanceRules: [
      (v) => !isNaN(v) || "La distancia debe ser un número",
      (v) => v > 0 || "La distancia debe ser mayor a 0",
    ],
    durationRules: [
      (v) => !isNaN(v) || "La duración debe ser un número",
      (v) => v > 0 || "La duración debe ser mayor a 0",
    ],
  }),
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Agregar Ruta" : "Editar Ruta";
    },
    imgedit() {
      return this.imgMiniatura;
    },
  },
  mounted() {
    this.initialize();
  },
  methods: {
    async showAdd() {
      try {
        const result = await handleRequest({
          endpoint: "location",
          method: "GET",
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.locations = result.data?.locations || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.locations = [];
        }
      } catch (error) {
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al procesar la solicitud.",
          3000
        );
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
      this.editedIndex = -1;
    },
    async initialize() {
      try {
        this.loading = true;
        const result = await handleRequest({
          endpoint: "route",
          method: "GET",
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.routes = result.data?.routes || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.routes = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
      } finally {
        this.loading = false;
      }
    },
    async save() {
      this.loading = true;
      if (this.editedIndex === -1) {

        this.valid = false;
        const fieldsToUpdate = [
          "name",
          "origin_id",
          "destination_id",
          "distance",
          "estimated",
          "status",
        ];

        let updatedFields = Object.keys(this.editedItem)
          .filter(
            (key) =>
              fieldsToUpdate.includes(key) &&
              this.editedItem[key] !== this.originalItem[key]
          )
          .reduce((obj, key) => {
            obj[key] = this.editedItem[key];
            return obj;
          }, {});
        if (Object.keys(updatedFields).length > 0) {
          try {
            const result = await handleRequest({
              endpoint: "route",
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
        }
      } else {
        this.valid = false;
        const fieldsToUpdate = [
          "name",
          "origin_id",
          "destination_id",
          "distance",
          "estimated",
          "status",
        ];
        let updatedFields = Object.keys(this.editedItem)
          .filter(
            (key) =>
              fieldsToUpdate.includes(key) &&
              this.editedItem[key] !== this.originalItem[key]
          )
          .reduce((obj, key) => {
            obj[key] = this.editedItem[key];
            return obj;
          }, {});
        if (Object.keys(updatedFields).length > 0) {
          updatedFields.id = this.editedItem.id;
          try {
            const result = await handleRequest({
              endpoint: "route",
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
      this.originalItem = Object.assign({}, item);
      this.editedItem = Object.assign({}, item);
      try {
        const result = await handleRequest({
          endpoint: "location",
          method: "GET",
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.locations = result.data?.locations || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.locations = [];
        }
      } catch (error) {
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al procesar la solicitud.",
          3000
        );
      } finally {
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
          endpoint: "route-destroy",
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
