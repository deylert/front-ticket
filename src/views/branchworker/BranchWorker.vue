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
        <div v-if="branch">
          <!-- Aquí puedes agregar más información sobre la sucursal -->
        </div>
      </v-col>
    </v-row>
  </v-snackbar>
  <v-container style="min-width: 100%; min-height: 100%">
    <v-card elevation="6" class="mx-2">
      <v-toolbar color="#1976D2">
        <span class="text-subtitle-2 ml-4"> Trabajadores de la Sucursal</span>
        <v-spacer></v-spacer>
        <v-btn class="text-subtitle-1 ml-12" color="#E7E9E9" variant="flat" @click="showAdd()">
          Agregar Trabajador
        </v-btn>
      </v-toolbar>

      <v-card-text>
        <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
          hide-details>
        </v-text-field>
        <v-data-table :headers="headers" :search="search" :items="branchworkers" class="elevation-1"
          style="max-height: 68vh; overflow-y: auto" :items-per-page-text="'Elementos por páginas'"
          no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
          <template v-slot:item.actions="{ item }">
            <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)" color="#1976D2" variant="tonal"
              elevation="1" title="Editar Trabajador"></v-btn>
            <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" color="#DA7171" variant="tonal"
              elevation="1" title="Eliminar Trabajador"></v-btn>
          </template>
          <template v-slot:item.workerName="{ item }">
            <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="large">
              <v-img :src="`${this.$axios.defaults.baseURL}images/${item.workerImage
                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
            {{ item.workerName }}
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </v-container>

  <v-dialog v-model="dialog" max-width="400px">
    <v-form ref="form" v-model="valid" enctype="multipart/form-data">
      <v-card>
        <v-toolbar color="#1976D2">
          <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
        </v-toolbar>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12" md="12">
                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.worker_id"
                  :items="workers" label="Personas" prepend-icon="mdi-account" item-title="name" item-value="id"
                  variant="underlined" :rules="selectRules" :disabled="this.editedIndex === 1">
                  <template v-slot:item="{ props, item }">
                    <v-list-item v-bind="props"
                      :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`"
                      :title="item.raw.name">
                      <v-list-item-subtitle class="d-flex flex-column">
                        <div>Correo: {{ item.raw.email }}</div>
                      </v-list-item-subtitle>
                    </v-list-item>
                  </template>
                </v-autocomplete>
              </v-col>
              <v-col cols="12" md="12">
                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.role_id" :items="roles"
                  label="Roles" prepend-icon="mdi-account-tie" item-title="name" item-value="id" variant="underlined"
                  :rules="selectRules">
                </v-autocomplete>
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
        <span class="text-subtitle-2 ml-4"> Eliminar Trabajdor</span>
      </v-toolbar>

      <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el Trabajador?</v-card-text>
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
  props: {
    branch: {
      type: Object,
      required: true,
      default: () => ({}), // Objeto vacío por defecto
    },
  },
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
    branchworkers: [],
    roles: [],
    workers: [],
    branch_id: "",
    data: {},
    headers: [
      { title: "Nombre del trabajdor", value: "workerName", width: "30%" },
      { title: "Rol", value: "roleName", width: "40%" },
      { title: "Acciones", value: "actions", sortable: false, width: "20%" },
    ],

    editedItem: {
      id: "",
      branch_id: "",
      role_id: "",
      worker_id: "",
    },
    originalItem: {
      id: "",
      branch_id: "",
      role_id: "",
      worker_id: "",
    },
    defaultItem: {
      id: "",
      branch_id: "",
      role_id: "",
      worker_id: "",
    },
    editedIndex: -1,
    search: "",
    selectRules: [(v) => !!v || "Seleccionar al menos un elemento"],
  }),
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Agregar Trabajador" : "Editar Trabajador";
    },
  },
  mounted() {
    this.branch_id = this.branch.id;
    this.initialize();
  },
  methods: {
    async showAdd() {
      this.data = {};
      this.data.branch_id = this.branch_id;
      this.data.type = 'Sucursal';
      try {
        const result = await handleRequest({
          endpoint: 'branch-workers-roles',
          method: 'POST',
          data: this.data
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.roles = result.data?.roles || [];
          this.workers = result.data?.workers.filter((worker) =>
            !this.branchworkers.some((branchworker) => branchworker.worker_id === worker.id)
          ) || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.roles = [];
          this.workers = [];
          this.showAlert('info', result.message || 'No hay datos disponibles.', 3000);
        }
      } catch (error) {
        this.showAlert('error', 'Ocurrió un error inesperado al cargar los datos.', 3000);
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
        this.data = {};
        this.data.branch_id = this.branch_id;
        const result = await handleRequest({
          endpoint: "branch-workers",
          method: "POST",
          data: this.data,
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.branchworkers = result.data?.branchWorkers || [];
          this.loading = false;
        } else {
          // Si no hay datos, asignamos un array vacío
          this.branchwrorkers = [];
          this.showAlert(
            "info",
            result.message || "No hay Trabajadores disponibles.",
            3000
          );
          this.loading = false;
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert(
          "error",
          "Ocurrió un error inesperado al cargar los trabajadores.",
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
        this.data = {};
        this.data.branch_id = this.branch_id;
        this.data.worker_id = this.editedItem.worker_id;
        this.data.role_id = this.editedItem.role_id;

        try {
          const result = await handleRequest({
            endpoint: "branch-worker",
            method: "POST",
            data: this.data,
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
        this.valid = false;
        const fieldsToUpdate = ["id", "branch_id", "role_id", "worker_id"];
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
              endpoint: "branch-worker",
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
      this.data = {};
      this.data.branch_id = this.branch_id;
      this.data.type = 'Sucursal';
      try {
        const result = await handleRequest({
          endpoint: 'branch-workers-roles',
          method: 'POST',
          data: this.data
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          this.roles = result.data?.roles || [];
          this.workers = result.data?.workers.filter((worker) =>
            !this.branchworkers.some((branchworker) => branchworker.worker_id === worker.id) ||
            worker.id === this.editedItem.worker_id
          ) || [];
        } else {
          // Si no hay datos, asignamos un array vacío
          this.roles = [];
          this.workers = [];
          this.showAlert('info', result.message || 'No hay datos disponibles.', 3000);
        }
      } catch (error) {
        this.showAlert('error', 'Ocurrió un error inesperado al cargar los datos.', 3000);
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
          endpoint: "branch-worker-destroy",
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
