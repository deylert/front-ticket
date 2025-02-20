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
  <v-toolbar color="#1976D2">
    <v-row align="left">
      <v-col cols="12" md="8" class="grow ml-4">
        <span class="text-subtitle-1"><strong>Estructuras de asientos</strong></span>
      </v-col>
    </v-row>
  </v-toolbar>
  <v-container>
    <!-- Botón para abrir el diálogo (alineado a la derecha) -->
    <div class="button-container">
      <v-btn @click="dialog = true" color="primary">Agregar Estructura</v-btn>
    </div>

    <!-- Diálogo para crear una nueva estructura -->
    <v-dialog v-model="dialog" max-width="600">
      <v-form ref="form" v-model="valid">
        <v-card>
          <v-toolbar color="#1976D2">
            <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
          </v-toolbar>
          <v-card-text>
            <v-row>
              <v-col cols="12" md="6">
                <v-text-field v-model="editedItem.name" clearable label="Nombre" prepend-icon="mdi-tag-outline"
                  variant="underlined" :rules="nameRules"></v-text-field>
              </v-col>
              <v-col cols="12" md="6">
                <!-- Campo para la cantidad de asientos -->
                <v-text-field v-model="editedItem.seatCount" label="Cantidad de Asientos" type="number"
                  @input="generateSeatMap" prepend-icon="mdi-seat" :rules="seatCountRules"
                  variant="underlined"></v-text-field>
              </v-col>
              <v-col cols="12" md="12">
                <v-textarea v-model="editedItem.description" clearable label="Descripción" prepend-icon="mdi-note"
                  variant="underlined"></v-textarea>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-card v-if="this.editedItem.seatMap.length > 0">
                  <v-card-text>
                    <div v-for="(row, rowIndex) in this.editedItem.seatMap" :key="rowIndex" class="seat-row">
                      <v-btn v-for="(seat, seatIndex) in row" :key="seatIndex"
                        :color="seat.selected ? 'primary' : 'grey'"
                        :disabled="selectedCount >= editedItem.seatCount && !seat.selected"
                        @click="toggleSeat(rowIndex, seatIndex)" class="seat-button">
                        {{ seat.label }}
                      </v-btn>
                    </div>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn @click="close" color="#DA7171" variant="flat">Cancelar</v-btn>
            <v-btn @click="save" color="#1976D2" variant="flat" :loading="this.loading"
              :disabled="!valid">Aceptar</v-btn>
          </v-card-actions>
        </v-card>
      </v-form>
    </v-dialog>

    <!-- Lista de estructuras guardadas -->
    <v-container fluid>
      <v-row>
        <v-col v-for="(structure, index) in structures" :key="index" cols="12" sm="6" md="4" lg="3">
          <v-card class="structure-card">
            <v-card-text class="card-content">
              <div class="structure-info">
                <div class="text-h6">{{ structure.name }}</div>
                <div class="text-body-1">{{ structure.description }}</div>
                <div class="text-caption">Asientos: {{ structure.seatCount }}</div>
              </div>

              <div v-for="(row, rowIndex) in structure.seatMap" :key="rowIndex" class="seat-row">
                <v-btn v-for="(seat, seatIndex) in row" :key="seatIndex" :color="seat.selected ? 'primary' : '#00000'"
                  class="seat-button-preview" disabled>
                  <v-icon v-if="seat.label">mdi-seat</v-icon>
                  {{ seat.label ? `${seat.label}` : '' }}
                </v-btn>
              </div>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <div class="structure-actions">
                <v-btn @click="deleteItem(structure)" color="#DA7171" variant="flat">Eliminar</v-btn>
                <v-btn @click="editStructure(structure)" color="#1976D2" variant="flat" class="ml-1">Editar</v-btn>
              </div>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>

      <!-- Botón para cargar más estructuras -->
      <div class="text-center mt-4">
        <v-btn v-if="hasMore" @click="initialize" :loading="loading" color="primary">
          Cargar más
        </v-btn>
        <p v-else>No hay más estructuras para mostrar.</p>
      </div>
    </v-container>
    <v-dialog v-model="dialogDelete" max-width="500px">
      <v-card>
        <v-toolbar color="#DA7171">
          <span class="text-subtitle-2 ml-4"> Eliminar una estructura</span>
        </v-toolbar>
        <v-card-text class="mt-2 mb-2"> ¿Desea eliminar la estructura?</v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="#DA7171" variant="flat" @click="closeDelete">Cancelar</v-btn>
          <v-btn color="#1976D2" variant="flat" :loading="loading" @click="deleteItemConfirm">Aceptar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { handleRequest } from "@/utils/api"; // Ruta al archivo
export default {
  data() {
    return {
      snackbar: false,
      sb_type: '',
      sb_message: '',
      sb_timeout: 2000,
      sb_title: '',
      sb_icon: '',
      valid: true,
      loading: false,
      dialogDelete: false,
      dialog: false, // Controla la visibilidad del diálogo
      editedItem: {
        id: '',
        name: '',
        description: '',
        seatCount: 0,
        seats: [],
        seatMap: [],
      },
      defaultItem: {
        id: '',
        name: '',
        description: '',
        seatCount: 0,
        seats: [],
        seatMap: [],
      },
      originalItem: {
        id: '',
        name: '',
        description: '',
        seatCount: 0,
        seats: [],
        seatMap: [],
      },
      nameRules: [
        (v) => !!v || "El campo es requerido",
        (v) => (v && v.length <= 50) ||
          "El campo debe tener menos de 51 caracteres",
        (v) => (v && v.length >= 3) ||
          "El campo debe tener al menos 3 caracteres",
      ],
      seatCountRules: [
        (v) => !!v || "El campo es requerido", // Asegura que no esté vacío
        (v) => (v > 0) || "La cantidad de asientos debe ser mayor a 0", // Asegura que sea mayor a 0
        (v) => Number.isInteger(Number(v)) || "Debe ser un número entero", // Asegura que sea un número
      ],
      data: {},
      editedIndex: -1,
      seatMap: [], // Matriz de asientos
      structures: [], // Array estático de estructuras
      selectedCount: 0, // Contador de asientos seleccionados
      nextSeatNumber: 1, // Siguiente número de asiento a asignar
      cursor: null,   // Cursor para la paginación
      hasMore: false, // Indica si hay más páginas
      isLoading: false, // Para evitar múltiples solicitudes simultáneas
    };
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? 'Agregar Estructura' : 'Editar Estructura';
    }
  },
  mounted() {
    this.initialize();
  },
  methods: {
    async loadStructures() {
      if (this.isLoading) return; // Evitar múltiples solicitudes

      this.isLoading = true;

      try {
        const response = await this.$axios.post('/api/structures', {
          limit: 10, // Número de estructuras por página
          cursor: this.cursor, // Cursor para la paginación
        });

        const { structures, hasMore, nextCursor } = response.data;

        // Agregar las nuevas estructuras a la lista
        this.structures = [...this.structures, ...structures];
        this.hasMore = hasMore;
        this.cursor = nextCursor; // Actualizar el cursor para la próxima página
      } catch (error) {
        console.error('Error al cargar las estructuras:', error);
      } finally {
        this.isLoading = false;
      }
    },
    // Genera la matriz de asientos con 4 columnas y el número mínimo de filas
    generateSeatMap() {
      const totalSeats = this.editedItem.seatCount;
      const columns = 4; // Número fijo de columnas
      const rows = Math.ceil(totalSeats / columns) + 1; // Número de filas
      this.seatMap = [];
      this.editedItem.seatMap = [];

      for (let i = 0; i < rows; i++) {
        const row = [];
        for (let j = 0; j < columns; j++) {
          row.push({ label: '', selected: false, disabled: false }); // Todas las celdas son seleccionables
        }
        this.seatMap.push(row);
        this.editedItem.seatMap.push(row);
      }

      this.selectedCount = 0; // Reinicia el contador de asientos seleccionados
      this.nextSeatNumber = 1; // Reinicia el contador de números de asiento
    },
    // Cambia el estado de selección de un asiento
    /*toggleSeat(rowIndex, seatIndex) {
      const seat = this.editedItem.seatMap[rowIndex][seatIndex];

      // Verificar si se puede seleccionar más asientos
      if (!seat.selected && this.selectedCount >= this.editedItem.seatCount) {
        return; // No se pueden seleccionar más asientos
      }

      // Cambiar el estado de selección del asiento
      seat.selected = !seat.selected;

      if (seat.selected) {
        // Asignar el siguiente número disponible
        seat.label = `${this.nextSeatNumber}`;
        this.nextSeatNumber++;
        this.selectedCount++;

        // Agregar el número del asiento seleccionado al array 'editedItem.seats'
        this.editedItem.seats.push(this.nextSeatNumber - 1); // Usamos nextSeatNumber-1 porque ya se incrementó
      } else {
        // Eliminar la etiqueta del asiento deseleccionado
        seat.label = '';
        this.nextSeatNumber--;
        this.selectedCount--;

        // Eliminar el número del asiento deseleccionado del array 'editedItem.seats'
        const index = this.editedItem.seats.indexOf(parseInt(seat.label));
        if (index !== -1) {
          this.editedItem.seats.splice(index, 1);
        }
      }
    },*/
    toggleSeat(rowIndex, seatIndex) {
      const seat = this.editedItem.seatMap[rowIndex][seatIndex];

      // Verificar si se puede seleccionar más asientos
      if (!seat.selected && this.selectedCount >= this.editedItem.seatCount) {
        return; // No se pueden seleccionar más asientos
      }

      // Cambiar el estado de selección del asiento
      seat.selected = !seat.selected;

      if (seat.selected) {
        // Asignar el siguiente número disponible
        seat.label = `${this.nextSeatNumber}`;
        this.nextSeatNumber++;
        this.selectedCount++;

        // Agregar el número del asiento seleccionado al array 'editedItem.seats'
        this.editedItem.seats.push(this.nextSeatNumber - 1); // Usamos nextSeatNumber-1 porque ya se incrementó
      } else {
        // Obtener el número del asiento deseleccionado
        const seatNumber = Number(seat.label);

        // Deseleccionar todos los asientos con un número mayor
        this.deselectFollowingSeats(seatNumber);

        // Eliminar la etiqueta del asiento deseleccionado
        seat.label = '';
        this.selectedCount--;

        // Eliminar el número del asiento deseleccionado del array 'editedItem.seats'
        const index = this.editedItem.seats.indexOf(seatNumber);
        if (index !== -1) {
          this.editedItem.seats.splice(index, 1);
        }

        // Actualizar el contador 'nextSeatNumber'
        this.nextSeatNumber = seatNumber;
      }
    },

    deselectFollowingSeats(seatNumber) {
      // Recorrer todos los asientos para deseleccionar los que tienen un número mayor
      this.editedItem.seatMap.forEach(row => {
        row.forEach(seat => {
          if (seat.selected && Number(seat.label) > seatNumber) {
            // Deseleccionar el asiento
            seat.selected = false;
            seat.label = '';
            this.selectedCount--;

            // Eliminar el número del asiento deseleccionado del array 'editedItem.seats'
            const index = this.editedItem.seats.indexOf(Number(seat.label));
            if (index !== -1) {
              this.editedItem.seats.splice(index, 1);
            }
          }
        });
      });
    },
    // Guarda la estructura en el array estático
    saveStructure() {
      const selectedSeats = this.seatMap
        .flat()
        .filter(seat => seat.selected)
        .map(seat => seat.label);
      const structure = {
        ...this.editedItem,
        seats: selectedSeats,
        seatMap: JSON.parse(JSON.stringify(this.seatMap)), // Guarda una copia de la matriz de asientos
      };
      this.editedItem.seatMap = JSON.parse(JSON.stringify(this.seatMap));
      this.structures.push(structure); // Agrega la estructura al array estático
      console.log(this.structures);
      this.dialog = false; // Cierra el diálogo
      this.resetForm(); // Reinicia el formulario
    },
    async initialize() {
      try {
        this.data = {};
        this.data.cursor = this.cursor;
        this.loading = true;
        const result = await handleRequest({
          endpoint: 'structure-cursor',
          method: 'POST',
          data: this.data
        });

        if (result.success) {
          // Si la solicitud es exitosa, asignamos las sucursales
          // Agregar las nuevas estructuras a la lista
          const { structures, hasMore, nextCursor } = result.data;

          // Agregar solo las estructuras nuevas a la lista
          this.structures.push(...structures);
          this.hasMore = hasMore;
          this.cursor = nextCursor; // Actualizar el cursor para la próxima página
        } else {
          // Si no hay datos, asignamos un array vacío
          this.structures = [];
        }
      } catch (error) {
        this.loading = false;
        // Captura de errores no controlados
        this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
      } finally {
        this.loading = false;
      }
    },
    close() {
      this.dialog = false;
      this.loading = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
      });
      this.editedIndex = -1;
      this.selectedCount = 0;
      this.nextSeatNumber = 1;
    },
    async save() {
      this.loading = true;
      this.data = {};
      if (this.editedIndex === -1) {
        this.valid = false;
        this.data.name = this.editedItem.name;
        this.data.description = this.editedItem.description;
        this.data.seatMap = JSON.parse(JSON.stringify(this.seatMap));
        this.data.seats = JSON.parse(JSON.stringify(this.editedItem.seats));
        this.data.seatCount = this.editedItem.seatCount;
        console.log('this.data');
        console.log(this.data);
        try {
          const result = await handleRequest({
            endpoint: 'structure',
            method: 'POST',
            data: this.data
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
        } finally {
          this.loading = false;
        }
      } else {
        const fieldsToUpdate = ['id', 'name', 'description', 'seats', 'seatCount', 'seatMap'];
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
              endpoint: 'structure',
              method: 'PUT',
              data: updatedFields
            });

            // Manejo de la respuesta según el resultado
            if (result.success) {
              this.showAlert("success", result.message, 3000);
              this.initialize();
            } else {
              this.loading = false;
              this.editedIndex = -1;
              this.showAlert("warning", result.message, 3000);
            }
          } catch (error) {
            this.loading = false;
            this.editedIndex = -1;
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
    // Método para abrir el formulario de edición con los datos de la estructura seleccionada
    editStructure(structure) {
      //this.close();

      // Clonar la estructura sin referencia
      this.editedItem = JSON.parse(JSON.stringify(structure));

      // Inicializar el seatMap con los asientos seleccionados
      this.editedItem.seatMap = this.editedItem.seatMap.map(row =>
        row.map(seat => ({
          ...seat,
          selected: seat.label && this.editedItem.seats.includes(Number(seat.label))
        }))
      );

      // Inicializar el contador de asientos seleccionados
      this.selectedCount = this.editedItem.seats.length;

      // Inicializar el siguiente número de asiento
      this.nextSeatNumber = this.selectedCount > 0
        ? Math.max(...this.editedItem.seats) + 1
        : 1;
      this.editedIndex = 1;
      // Abrir el diálogo
      this.dialog = true;
    },

    // Reinicia el formulario
    resetForm() {
      this.editedItem = {
        name: '',
        seatCount: 0,
        seats: [],
      };
      this.seatMap = [];
      this.selectedCount = 0;
      this.nextSeatNumber = 1;
    },
    deleteItem(item) {
      this.editedIndex = -1;
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
          endpoint: 'structure-destroy',
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

<style>
/* Alinear el botón a la derecha */
.button-container {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 20px;
}

.seat-button {
  margin: 5px;
  /* Espacio entre los asientos */
  border-radius: 8px;
  /* Bordes redondeados */
  min-width: 45px;
  /* Tamaño mínimo */
  min-height: 45px;
  /* Tamaño mínimo */
  font-size: 14px;
  /* Tamaño de la fuente */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  /* Sombra suave */
}

/* Estilo para asientos seleccionados */
.v-btn--active.seat-button {
  background-color: #1976d2;
  /* Color primario */
  color: white;
}

/* Estilo para asientos no seleccionados */
.v-btn.seat-button {
  background-color: #e0e0e0;
  /* Color gris */
  color: #000;
}

/* Estilos para la vista previa de la matriz de asientos */
.seat-map-preview {
  margin-top: 10px;
}

.seat-button-preview {
  margin: 2px;
  /* Espacio entre los asientos */
  border-radius: 4px;
  /* Bordes redondeados */
  min-width: 25px;
  /* Tamaño mínimo */
  min-height: 25px;
  /* Tamaño mínimo */
  font-size: 12px;
  /* Tamaño de la fuente */
}

.structure-card {
  height: 100%;
  /* Asegura que todas las cards tengan la misma altura */
  width: 100%;
  /* Asegura que todas las cards tengan la misma altura */
  display: flex;
  flex-direction: column;
}

.card-content {
  flex: 1;
  /* Hace que el contenido ocupe todo el espacio disponible */
  display: flex;
  flex-direction: column;
}

.structure-info {
  margin-bottom: 16px;
  /* Espacio entre los datos y la matriz */
}

.seat-map-preview {
  flex: 1;
  /* Hace que la matriz ocupe el espacio restante */
  display: flex;
  flex-direction: column;
  justify-content: center;
  /* Centra la matriz verticalmente */
}

.seat-row {
  display: flex;
  justify-content: center;
  /* Centra los asientos horizontalmente */
  margin-bottom: 8px;
  /* Espacio entre filas de asientos */
}

.structure-actions {
  margin-top: auto;
  /* Empuja los botones hacia la parte inferior */
  display: flex;
  justify-content: flex-end;
  /* Alinea los botones a la derecha */
}

/* Estilos para la vista previa de la matriz de asientos */
.seat-map-preview {
  margin-bottom: 10px;
  flex-grow: 1;
  /* Hace que la matriz ocupe el espacio disponible */
}

.seat-button-preview {
  margin: 1px;
  /* Espacio entre los asientos */
  border-radius: 4px;
  /* Bordes redondeados */
  min-width: 30px;
  /* Tamaño mínimo */
  min-height: 30px;
  /* Tamaño mínimo */
  font-size: 10px;
  /* Tamaño de la fuente */
}

/* Estilos para los botones de acciones */
.structure-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: auto;
  /* Alinear los botones al final */
}
</style>