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
                <div v-if="vehicle">
                    <!-- Aquí puedes agregar más información sobre la sucursal -->
                </div>
            </v-col>
        </v-row>
    </v-snackbar>
    <v-container style="min-width: 100%; min-height: 100%">
        <v-card elevation="6" class="mx-2">
            <v-toolbar :color="paleteColors.primary">
                <span class="text-subtitle-2 ml-4"> Trabajadores asociados al vehículo:</span>
                <span class="text-subtitle-2 ml-4">
                    <!-- Avatar del vehículo -->
                    <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                        <v-img :src="`${this.$axios.defaults.baseURL}images/${this.vehicle.image}?t=${Date.now()}`"
                            alt="image"></v-img>
                    </v-avatar>
                     {{ this.vehicle.plate }}
                </span>
                <v-spacer></v-spacer>
                <v-btn class="text-subtitle-1 ml-12" :color="paleteColors.white" variant="tonal" elevation="2" @click="showAdd()">
                    Agregar Trabajador
                </v-btn>
            </v-toolbar>

            <v-card-text>
                <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
                    hide-details>
                </v-text-field>
                <v-data-table :headers="headers" :search="search" :items="vehicleworkers" class="elevation-1"
                    style="max-height: 68vh; overflow-y: auto" :items-per-page-text="'Elementos por páginas'"
                    no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
                    <template v-slot:item.actions="{ item }">
                        <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" :color="paleteColors.error"
                            variant="tonal" elevation="1" title="Eliminar Trabajador"></v-btn>
                    </template>
                    <template v-slot:item.name="{ item }">
                        <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="large">
                            <v-img :src="`${this.$axios.defaults.baseURL}images/${item.name
                                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar><!--+'?$'+Date.now()-->
                        {{ item.name }}
                    </template>
                </v-data-table>
            </v-card-text>
        </v-card>
    </v-container>

    <v-dialog v-model="dialog" max-width="400px">
        <v-form ref="form" v-model="valid" enctype="multipart/form-data">
            <v-card>
                <v-toolbar :color="paleteColors.primary">
                    <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
                </v-toolbar>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="12" md="12">
                                <v-autocomplete :no-data-text="'No hay datos disponibles'"
                                    v-model="editedItem.worker_id" :items="workers" label="Trabajadores"
                                    prepend-icon="mdi-account" item-title="name" item-value="id" variant="underlined"
                                    :rules="selectRules">
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
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn :color="paleteColors.gris" variant="flat" @click="close">Cancelar</v-btn>
                    <v-btn :color="paleteColors.primary" variant="flat" @click="save" :disabled="!valid"
                        :loading="loading">Aceptar</v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>
    <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
            <v-toolbar :color="paleteColors.error">
                <span class="text-subtitle-2 ml-4"> Eliminar Trabajador</span>
            </v-toolbar>

            <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el Trabajador seleccionado?</v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn :color="paleteColors.gris" variant="flat" @click="closeDelete"> Cancelar </v-btn>
                <v-btn :color="paleteColors.error" variant="flat" @click="deleteItemConfirm"> Aceptar </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { paleteColors } from "@/assets/colors";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
export default {
    props: {
        vehicle: {
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
        paleteColors: paleteColors,
        valid: true,
        loading: false,
        mostrar: false,

        dialog: false,
        dialogDelete: false,
        vehicleworkers: [],
        workers: [],
        vehicle_id: "",
        data: {},
        headers: [
            { title: "Nombre", value: "name", width: "40%" },
            { title: "email", value: "email", width: "40%" },
            { title: "Acciones", value: "actions", sortable: false, width: "20%" },
        ],

        editedItem: {
            id: "",
            vehicle_id: "",
            worker_id: "",
        },
        originalItem: {
            id: "",
            vehicle_id: "",
            worker_id: "",
        },
        defaultItem: {
            id: "",
            vehicle_id: "",
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
        this.vehicle_id = this.vehicle.id;
        this.initialize();
    },
    methods: {
        async showAdd() {
            try {
                const result = await handleRequest({
                    endpoint: 'worker',
                    method: 'GET'
                });

                if (result.success) {
                    this.workers = result.data?.workers.filter((worker) =>
                        !this.vehicleworkers.some((vehicleworker) => vehicleworker.worker_id === worker.id)
                    ) || [];
                } else {
                    this.workers = [];
                }
            } catch (error) {
                this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
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
                this.data.vehicle_id = this.vehicle_id;
                const result = await handleRequest({
                    endpoint: "vehicle-workers",
                    method: "POST",
                    data: this.data,
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.vehicleworkers = result.data?.vehicleWorkers || [];
                    this.loading = false;
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.vehicleworkers = [];
                    this.loading = false;
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
                this.data = {};
                this.data.vehicle_id = this.vehicle_id;
                this.data.worker_id = this.editedItem.worker_id;

                try {
                    const result = await handleRequest({
                        endpoint: "vehicle-worker",
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
                const fieldsToUpdate = ["id", "vehicle_id", "worker_id"];
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
                            endpoint: "vehicle-worker",
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
                    endpoint: 'worker',
                    method: 'GET'
                });

                if (result.success) {
                    this.workers = result.data?.workers.filter((worker) =>
                        !this.vehicleworkers.some((vehicleworker) => vehicleworker.worker_id === worker.id) ||
                        worker.id === this.editedItem.worker_id
                    ) || [];
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
                    endpoint: "vehicle-worker-destroy",
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