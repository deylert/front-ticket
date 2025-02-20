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
    <v-container fluid fill-height>
        <v-card elevation="6" class="mx-2">
            <v-toolbar color="#1976D2">
                <v-row align="center">
                    <v-col cols="12" md="8" class="grow ml-4">
                        <span class="text-subtitle-1"><strong>Listado de Permisos</strong></span>
                    </v-col>
                    <v-col cols="12" md="3" class="text-right">
                        <v-btn class="text-subtitle-1 ml-12" color="white" variant="tonal" elevation="2"
                            prepend-icon="mdi-plus-circle" @click="showAdd">
                            Agregar Permiso
                        </v-btn>
                    </v-col>
                </v-row>
            </v-toolbar>

            <v-card-text>
                <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
                    hide-details>
                </v-text-field>
                <v-data-table :headers="headers" :search="search" :items="permisions" class="elevation-1"
                    style="max-height: 68vh; overflow-y: auto;" :items-per-page-text="'Elementos por páginas'"
                    no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
                    <template v-slot:item.actions="{ item }">
                        <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)" color="#1976D2"
                            variant="tonal" elevation="1" title="Editar Permiso"></v-btn>
                        <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" color="#DA7171"
                            variant="tonal" elevation="1" title="Eliminar Permiso"></v-btn>
                    </template>
                </v-data-table>
            </v-card-text>
        </v-card>
    </v-container>

    <v-dialog v-model="dialog" max-width="600px">
        <v-form ref="form" v-model="valid">
            <v-card>
                <v-toolbar color="#1976D2">
                    <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
                </v-toolbar>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="12" md="6">
                                <v-text-field v-model="editedItem.name" clearable label="Nombre"
                                    prepend-icon="mdi-tag-outline" variant="underlined"
                                    :rules="nameRules"></v-text-field>
                            </v-col>
                            <v-col cols="12" md="6">
                                    <v-text-field v-model="editedItem.module" clearable label="Módulo"
                                        prepend-icon="mdi-view-module" variant="underlined"></v-text-field>
                            </v-col>
                            <v-col cols="12" md="12">
                                <v-textarea v-model="editedItem.description" clearable label="Descripción"
                                    prepend-icon="mdi-note" variant="underlined"></v-textarea>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="#DA7171" variant="flat" @click="close">Cancelar</v-btn>
                    <v-btn color="#1976D2" variant="flat" :loading="loading" @click="save"
                        :disabled="!valid">Aceptar</v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>

    <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
            <v-toolbar color="#DA7171">
                <span class="text-subtitle-2 ml-4"> Eliminar un permiso</span>
            </v-toolbar>
            <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el permiso?</v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="#DA7171" variant="flat" @click="closeDelete">Cancelar</v-btn>
                <v-btn color="#1976D2" variant="flat" :loading="loading" @click="deleteItemConfirm">Aceptar</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { handleRequest } from "@/utils/api"; // Ruta al archivo
export default {
    data: () => ({
        snackbar: false,
        sb_type: '',
        sb_message: '',
        sb_timeout: 2000,
        sb_title: '',
        sb_icon: '',
        valid: true,
        loading: false,
        dialog: false,
        dialogDelete: false,
        permisions: [],
        data: {},
        headers: [
            { title: 'Nombre', value: 'name', width: '30%' },
            { title: 'Módulo', value: 'module', width: '10%' },
            { title: 'Descripción', value: 'description', width: '40%' },
            { title: 'Acciones', value: 'actions', sortable: false, width: '20%' },
        ],

        editedItem: {
            id: '',
            name: '',
            description: '',
            module: '',
        },
        defaultItem: {
            id: '',
            name: '',
            description: '',
            module: '',
        },
        originalItem: {
            id: '',
            name: '',
            description: '',
            module: '',
        },
        editedIndex: -1,
        search: '',
        nameRules: [
            (v) => !!v || "El campo es requerido",
            (v) => (v && v.length <= 50) ||
                "El campo debe tener menos de 51 caracteres",
            (v) => (v && v.length >= 3) ||
                "El campo debe tener al menos 3 caracteres",
        ],
        selectRules: [(v) => !!v || "Seleccionar al menos un elemento"],
    }),
    computed: {
        formTitle() {
            return this.editedIndex === -1 ? 'Agregar Permiso' : 'Editar Permiso';
        }
    },
    mounted() {
        this.initialize();
    },
    methods: {
        showAdd() {
            this.editedIndex = -1
            this.dialog = true;
        },
        close() {
            this.dialog = false;
            this.loading = false;
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
                    endpoint: 'permission',
                    method: 'GET',
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.permisions = result.data?.permissions; // Si no hay permissions, asigna un arreglo vacío
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.permissions = [];
                }
            } catch (error) {
                this.loading = false;
                // Captura de errores no controlados
                this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
            } finally {
                this.loading = false;
            }
        },
        async save() {
            this.loading = true;
            if (this.editedIndex === -1) {
                this.valid = false;
                const fieldsToUpdate = ['name', 'module', 'description'];

        let updatedFields = Object.keys(this.editedItem)
          .filter((key) => fieldsToUpdate.includes(key) && this.editedItem[key] !== this.originalItem[key])
          .reduce((obj, key) => {
            obj[key] = this.editedItem[key];
            return obj;
          }, {});
        if (Object.keys(updatedFields).length > 0) {
          try {
            const result = await handleRequest({
              endpoint: 'permission',
              method: 'POST',
              data: updatedFields
            });

            // Manejo de la respuesta según el resultado
            if (result.success) {
              this.loading = false;
              this.showAlert("success", result.message, 3000);
              this.initialize();
            } else {
              this.loading = false;
              this.showAlert("warning", result.message, 3000);
            }
          } catch (error) {
            this.loading = false;
            // Este bloque captura errores inesperados fuera del manejo estándar
            this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
          }
        } else {
          this.loading = false;
          this.showAlert("success", "Debe completar los datos.", 3000);
        }
            } else {
                const fieldsToUpdate = ['id', 'name', 'module', 'description'];
                let updatedFields = Object.keys(this.editedItem)
                    .filter((key) => fieldsToUpdate.includes(key) && this.editedItem[key] !== this.originalItem[key])
                    .reduce((obj, key) => {
                        obj[key] = this.editedItem[key];
                        return obj;
                    }, {});
                if (Object.keys(updatedFields).length > 0) {
                    updatedFields.id = this.editedItem.id;
                    this.loading = true;
                    try {
                        const result = await handleRequest({
                            endpoint: 'permission',
                            method: 'PUT',
                            data: updatedFields
                        });

                        // Manejo de la respuesta según el resultado
                        if (result.success) {
                            this.showAlert("success", result.message, 3000);
                            this.initialize();
                        } else {
                            this.loading = false;
                            this.showAlert("warning", result.message, 3000);
                        }
                    } catch (error) {
                        this.loading = false;
                        // Este bloque captura errores inesperados fuera del manejo estándar
                        this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
                    }
                } else {
                    this.loading = false;
                    this.showAlert("success", "No se realizaron cambios.", 3000);
                }
            }
            this.close();
        },
        editItem(item) {
            this.editedIndex = 1;
            this.originalItem = Object.assign({}, item);
            this.editedItem = Object.assign({}, item);
            this.dialog = true;
        },
        deleteItem(item) {
            this.editedIndex = 1;
            this.editedItem.id = item.id;
            this.dialogDelete = true;
        },
        closeDelete() {
            this.dialogDelete = false
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem)
            })
        },
        async deleteItemConfirm() {
            this.loading = true;
            try {
                let request = {
                    id: this.editedItem.id
                };
                const result = await handleRequest({
                    endpoint: 'permission-destroy',
                    method: 'POST',
                    data: request
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
                this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
            } finally {
                this.loading = false;
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
.avatar-border {
    border: 2px solid #000;
    /* Aquí se define el borde */
}
</style>