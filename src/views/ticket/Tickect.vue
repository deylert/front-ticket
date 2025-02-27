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
    <v-container style="min-width: 100%; min-height: 100%;">
        <v-card elevation="6" class="mx-2">
            <v-toolbar color="#1976D2">
                <v-row align="center">
                    <v-col cols="12" md="8" class="grow ml-4">
                        <span class="text-subtitle-1"><strong>Ticket Vendidos</strong></span>
                    </v-col>
                    <v-col cols="12" md="3" class="text-right">
                        <v-btn class="text-subtitle-1 ml-12" color="white" variant="tonal" elevation="2"
                            prepend-icon="mdi-plus-circle" @click="showAdd">
                            Agregar Ticket
                        </v-btn>
                    </v-col>
                </v-row>
            </v-toolbar>

            <v-card-text>
                <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar" single-line
                    hide-details>
                </v-text-field>

                <v-data-table :headers="headers" :search="search" :items="tickets" class="elevation-1"
                    style="max-height: 65vh; overflow-y: auto;" :items-per-page-text="'Elementos por páginas'"
                    no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
                    <template v-slot:item.actions="{ item }">
                        <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)" color="#1976D2"
                            variant="tonal" elevation="1" title="Editar Ticket"></v-btn>
                        <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)" color="#DA7171"
                            variant="tonal" elevation="1" title="Eliminar Ticket"></v-btn>
                    </template>
                    <template v-slot:item.tripOrigin="{ item }">
                        <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                            <v-img :src="`${this.$axios.defaults.baseURL}images/${item.originImage
                                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar>
                        {{ item.tripOrigin }}
                    </template>
                    <template v-slot:item.tripDestination="{ item }">
                        <v-avatar class="mr-1" elevation="3" color="grey-lighten-4" size="small">
                            <v-img :src="`${this.$axios.defaults.baseURL}images/${item.destinationImage
                                }?t=${Date.now()}`" alt="image"></v-img> </v-avatar>
                        {{ item.tripDestination }}
                    </template>
                </v-data-table>
            </v-card-text>
        </v-card>
    </v-container>
    <v-dialog v-model="dialog" max-width="700px">
        <v-form ref="form" v-model="valid" enctype="multipart/form-data">
            <v-card>
                <v-toolbar color="#1976D2">
                    <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
                </v-toolbar>
                <v-card-text>
                    <v-container>
                        <v-row style="margin-top: 5px">
                            <!-- Selección de viaje -->
                            <v-col cols="12" md="12">
                                <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.trip_id"
                                    :items="trips" label="Viaje" prepend-icon="mdi-road" item-title="name"
                                    item-value="id" variant="underlined" :rules="selectRules" density="compact"
                                    @update:model-value="updateSeats">
                                    <template v-slot:item="{ props, item }">
                                        <div>
                                            <v-list-item v-bind="props">
                                                <v-list-item-content>
                                                    <v-row align="center" no-gutters>
                                                        <!-- Origen -->
                                                        <v-col cols="12" md="6" class="d-flex align-center">
                                                            <v-avatar>
                                                                <v-img
                                                                    :src="`${this.$axios.defaults.baseURL}images/${item.raw.originImage}`"
                                                                    max-width="40" />
                                                            </v-avatar>
                                                            <div class="ml-2 text-truncate" :title="item.raw.origin"
                                                                style="max-width: 200px;">
                                                                {{ item.raw.origin }}
                                                            </div>
                                                        </v-col>

                                                        <!-- Destino -->
                                                        <v-col cols="12" md="6" class="d-flex align-center">
                                                            <v-avatar>
                                                                <v-img
                                                                    :src="`${this.$axios.defaults.baseURL}images/${item.raw.destinationImage}`"
                                                                    max-width="40" />
                                                            </v-avatar>
                                                            <div class="ml-2 text-truncate"
                                                                :title="item.raw.destination" style="max-width: 200px;">
                                                                {{ item.raw.destination }}
                                                            </div>
                                                        </v-col>
                                                    </v-row>

                                                    <!-- Horario y Llegada -->
                                                    <v-row align="center" no-gutters>
                                                        <v-col cols="auto" class="d-flex align-center">
                                                            <div class="text-truncate" style="max-width: 150px;">
                                                                <strong>Salida:</strong> {{ item.raw.schedule }}
                                                            </div>
                                                        </v-col>
                                                        <v-col cols="auto" class="d-flex align-center ml-4">
                                                            <div class="text-truncate" style="max-width: 150px;">
                                                                <strong>Llegada:</strong> {{ item.raw.arrival }}
                                                            </div>
                                                        </v-col>
                                                        <v-col cols="auto" class="d-flex align-center ml-4">
                                                            <v-avatar>
                                                                <v-img
                                                                    :src="`${this.$axios.defaults.baseURL}images/${item.raw.imageVehicle}`"
                                                                    max-width="40" />
                                                            </v-avatar>
                                                            <div class="ml-2 text-truncate" :title="item.raw.plate"
                                                                style="max-width: 200px;">
                                                                {{ item.raw.plate }}
                                                            </div>
                                                        </v-col>
                                                    </v-row>
                                                </v-list-item-content>
                                            </v-list-item>
                                        </div>
                                        <v-divider></v-divider>
                                    </template>
                                </v-autocomplete>
                            </v-col>

                            <!-- Método de pago -->
                            <v-col cols="12" md="6">
                                <v-select v-model="editedItem.method" :items="paymentMethods" label="Método de pago"
                                    item-value="value" item-title="text" variant="underlined" density="compact"
                                    :rules="[(v) => !!v || 'Seleccione un método de pago']"
                                    prepend-icon="mdi-cash"></v-select>
                            </v-col>

                            <!-- Fecha -->
                            <v-col cols="12" md="6">
                                <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
                                    transition="scale-transition" offset-y min-width="190px">
                                    <template v-slot:activator="{ props }">
                                        <v-text-field v-bind="props" :modelValue="dateFormatted" variant="underlined"
                                            prepend-icon="mdi-calendar" label="Fecha" density="compact"></v-text-field>
                                    </template>
                                    <v-locale-provider locale="es">
                                        <v-date-picker header="Calendario" title="Seleccione la fecha" color="#1976D2"
                                            :modelValue="input" @update:model-value="updateDate" format="yyyy-MM-dd"
                                            :min="new Date().toISOString().split('T')[0]"></v-date-picker>
                                    </v-locale-provider>
                                </v-menu>
                            </v-col>

                            <!-- Precio del pasaje -->
                            <v-col cols="12" md="4">
                                <v-text-field v-model="editedItem.price" label="Precio del pasaje" type="number"
                                    variant="underlined" density="compact" prepend-icon="mdi-cash"
                                    :rules="[(v) => v > 0 || 'Debe ser un precio válido']"
                                    placeholder="Ingrese el precio del pasaje" min="0" step="0.01"
                                    readonly></v-text-field>
                            </v-col>

                            <!-- Cantidad de pasajes -->
                            <v-col cols="12" md="4">
                                <v-text-field v-model="editedItem.quantity" label="Cantidad de pasajes" type="number"
                                    variant="underlined" density="compact" prepend-icon="mdi-ticket"
                                    placeholder="Ingrese la cantidad" min="1" @update:model-value="calculateTotal"
                                    :rules="quantityAndPassengerRules" :disabled="!editedItem.trip_id || !aviable"
                                    :hint="!editedItem.quantity ? `Asientos disponibles: ${aviable}` : ''"
                                    persistent-hint></v-text-field>
                            </v-col>

                            <!-- Selección de asientos -->
                            <v-col cols="12" md="4" v-if="editedItem.quantity">
                                <v-row>
                                    <v-menu v-model="showSeatsMenu" activator="parent" offset-y
                                        :close-on-content-click="false" :close-on-click-outside="false"
                                        :close-on-back="false">
                                        <template v-slot:activator="{ props }">
                                            <v-text-field v-bind="props" ref="seatsField"
                                                :value="selectedSeats.length > 0 ? selectedSeats.join(', ') : 'Seleccionar Asientos'"
                                                color="primary" dark readonly style="text-transform: none"
                                                :disabled="editedItem.quantity <= 0" prepend-icon="mdi-seat"
                                                variant="underlined"
                                                :rules="[v => selectedSeats.length > 0 || 'Debe seleccionar al menos un asiento']"></v-text-field>
                                        </template>

                                        <!-- Contenido del menú -->
                                        <v-card style="max-width: 300px">
                                            <v-card-title class="text-h6">Seleccione sus asientos</v-card-title>
                                            <v-card-text>
                                                <v-row>
                                                    <!-- Mostrar asientos en filas de 2 -->
                                                    <!-- Mostrar asientos en filas de 2 -->
                                                    <v-col cols="12" class="d-flex align-center justify-center">
                                                        <div class="seat-map-preview"
                                                            style="display: flex; flex-direction: column;">
                                                            <div v-for="(row, rowIndex) in seatMap" :key="rowIndex"
                                                                class="seat-row"
                                                                style="display: flex; flex-direction: row;">
                                                                <template v-for="(seat, seatIndex) in row"
                                                                    :key="seatIndex">
                                                                    <v-btn v-if="seat.selected"
                                                                        :color="getSeatColor(seat)"
                                                                        class="seat-button-preview"
                                                                        :disabled="!isSeatAvailable(seat)"
                                                                        @click="toggleSeat(seat)"
                                                                        style="min-width: 30px; min-height: 30px; font-size: 0.8rem; font-weight: bold;">
                                                                        <!-- Ícono de asiento -->
                                                                        <v-icon v-if="seat.label">mdi-seat</v-icon>
                                                                        {{ Number(seat.label) ? `${Number(seat.label)}`
                                                                            : '' }}
                                                                    </v-btn>
                                                                </template>
                                                            </div>
                                                        </div>
                                                    </v-col>
                                                </v-row>

                                                <!-- Mensaje de error si se seleccionan demasiados asientos -->
                                                <v-alert v-if="selectedSeats.length != editedItem.quantity" type="error"
                                                    class="mt-3">
                                                    Debe Seleccionar {{ editedItem.quantity }} asiento(s).
                                                </v-alert>
                                            </v-card-text>
                                            <v-card-actions>
                                                <v-spacer></v-spacer>
                                                <!-- Botón para cerrar el menú -->
                                                <v-btn variant="text" @click="showSeatsMenu = false"
                                                    :disabled="Number(selectedSeats.length) !== Number(editedItem.quantity)">
                                                    Cerrar
                                                </v-btn>
                                            </v-card-actions>
                                        </v-card>
                                    </v-menu>
                                </v-row>
                            </v-col>

                            <!-- Pasajeros adultos y menores -->
                            <v-row>
                                <v-col cols="12" md="4">
                                    <v-text-field v-model="editedItem.adults" label="Pasajeros adultos" type="number"
                                        variant="underlined" density="compact" prepend-icon="mdi-account"
                                        placeholder="Ingrese la cantidad de adultos" min="0"
                                        @update:model-value="calculateTotal"
                                        :rules="quantityAndPassengerRules"></v-text-field>
                                </v-col>

                                <v-col cols="12" md="4">
                                    <v-text-field v-model="editedItem.minors" label="Pasajeros menores" type="number"
                                        variant="underlined" density="compact" prepend-icon="mdi-account-child"
                                        placeholder="Ingrese la cantidad de menores" min="0"
                                        @update:model-value="calculateTotal"
                                        :rules="quantityAndPassengerRules"></v-text-field>
                                </v-col>

                                <v-col cols="12" md="4">
                                    <v-text-field v-model="editedItem.total" label="Total a pagar" type="number"
                                        variant="underlined" density="compact" prepend-icon="mdi-cash"
                                        readonly></v-text-field>
                                </v-col>
                            </v-row>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="#DA7171" variant="flat" @click="close">Cancelar</v-btn>
                    <v-btn color="#1976D2" variant="flat" @click="save"
                        :disabled="!valid || Number(selectedSeats.length) !== Number(editedItem.quantity)"
                        :loading="loading">Aceptar</v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>
    <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
            <v-toolbar color="#DA7171">
                <span class="text-subtitle-2 ml-4"> Eliminar un Ticket</span>
            </v-toolbar>

            <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el ticket?</v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="#DA7171" variant="flat" @click="closeDelete"> Cancelar </v-btn>
                <v-btn color="#1976D2" variant="flat" @click="deleteItemConfirm" :loading="loading"> Aceptar </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import _ from 'lodash';
import { format } from 'date-fns';
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
        branch_id: '',
        trips: [],
        routes: [],
        vehicles: [],
        workers: [],
        tickets: [],
        data: {},
        hasStartedSelecting: false,
        seats: 0, // Ejemplo de asientos disponibles
        selectedSeats: [], // Aquí se almacenan los asientos seleccionados
        reservedSeats: [],
        availableSeats: [],
        aviable: '',
        branches: [],
        showSeatsMenu: false,
        headers: [
            { title: "Ruta", value: "tripName", },
            { title: "Origen", value: "tripOrigin", },
            { title: "Destino", value: "tripDestination", },
            { title: "Fecha", value: "date", },
            { title: "Horario", value: "schedule", },
            { title: "Metodo", value: "method", },
            { title: "Pasajes", value: "quantity", },
            { title: "Adultos", value: "adults", },
            { title: "Menores", value: "minors", },
            { title: "Asientos", value: "seats", },
            { title: "Precio", value: "price", },
            { title: "Total", value: "total", },
            { title: "Acciones", value: "actions", sortable: false, width: "10%" },
        ],

        editedItem: {
            id: "",
            trip_id: "",
            branch_id: "",
            user_id: "",
            date: "",
            method: "",
            status: "",
            quantity: "",
            price: "",
            total: "",
            adults: "",
            minors: "",
            seats: [],
        },
        originalItem: {
            id: "",
            trip_id: "",
            branch_id: "",
            user_id: "",
            date: "",
            method: "",
            status: "",
            quantity: "",
            price: "",
            total: "",
            seats: [],
            adults: "",
            minors: "",
        },
        defaultItem: {
            id: "",
            trip_id: "",
            branch_id: "",
            user_id: "",
            date: "",
            method: "",
            status: "",
            quantity: "",
            price: "",
            total: "",
            seats: [],
            adults: "",
            minors: "",
        },
        paymentMethods: [
            { text: "Efectivo", value: "Efectivo" },
            { text: "Débito", value: "Debito" },
            { text: "Crédito", value: "Credito" },
        ],
        editedIndex: -1,
        search: "",
        menu: false,
        menu2: false,
        input: null,
        input2: null,
        tab: null,
        route: '',
        seatMap: [],
        nameRules: [
            (v) => !!v || "El campo es requerido",
            (v) => (v && v.length <= 50) || "El campo debe tener menos de 51 caracteres",
            (v) => (v && v.length >= 3) || "El campo debe tener al menos de 3 caracteres",
        ],
        selectRules: [(v) => !!v || "Seleccionar al menos un elemento"],
        //prueba borrar  
        currentPage: 1, // Página actual
        itemsPerPage: 6, // Elementos por página

    }),
    computed: {
        formTitle() {
            return this.editedIndex === -1 ? "Agregar Ticket" : "Editar Ticket";
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
        evenSeats() {
            return this.availableSeats.filter((seat, index) => index % 2 === 0);
        },
        oddSeat() {
            return this.availableSeats.length % 2 !== 0 ? this.availableSeats[this.availableSeats.length - 1] : null;
        },
        quantityAndPassengerRules() {
            return [
                () => {
                    if (!this.editedItem.quantity || this.editedItem.quantity <= 0) {
                        return "La cantidad de pasajes debe ser mayor a cero.";
                    }
                    //alert(this.availableSeats.length);
                    const availableSeats = this.availableSeats.length;

                    if (this.editedItem.quantity > availableSeats) {
                        return `La cantidad de pasajes no puede ser mayor a los asientos disponibles (${availableSeats}).`;
                    }

                    if (this.validateQuantity()) {
                        return true;
                    }
                    return "La suma de adultos y menores no puede ser mayor que la cantidad de pasajes.";
                },
            ];
        },
    },
    watch: {
        selectedSeats(newValue) {
            if (typeof newValue === 'string') {
                this.selectedSeats = JSON.parse(newValue);
            }
        },
        'editedItem.quantity'(newValue) {
            this.editedItem.adults = Math.min(this.editedItem.adults, newValue);
            this.editedItem.minors = Math.min(this.editedItem.minors, newValue);
            this.$refs.form.validate();
        },
    },
    mounted() {
        this.role = JSON.parse(LocalStorageService.getItem('role'));
        if (this.role === 'Administrador') {
            this.showBranches();
        } else {
            this.branch_id = LocalStorageService.getItem('branch_id');
        }
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
                }
            } catch (error) {
                // Captura de errores no controlados
                this.showAlert('error', 'Ocurrió un error inesperado al procesar la solicitud.', 3000);
            } finally {
                this.loading = false;
                this.initialize();
            }
        },
        getSeatColor(seat) {
            if (this.isSeatReserved(seat.label)) {
                return 'red'; // Asiento reservado
            } else if (this.selectedSeats.includes(Number(seat.label))) {
                return 'primary'; // Asiento seleccionado
            } else {
                return 'green'; // Asiento disponible
            }
        },
        isSeatAvailable(seat) {
            return seat.label && !this.isSeatReserved(seat.label);
        },
        updateSeats(tripId) {
            const selectedTrip = this.trips.find((trip) => trip.id === tripId);

            if (selectedTrip) {
                this.seats = selectedTrip.seats;
                this.editedItem.price = selectedTrip.price;
                this.reservedSeats = selectedTrip.reservedSeats;
                this.seatMap = selectedTrip.seatMap;

                // Generar asientos disponibles y reservados
                this.availableSeats = this.generateAvailableSeats(this.seatMap, this.reservedSeats);
                this.aviable = this.availableSeats.length;
                console.log('this.aviable');
                console.log(this.aviable);

            } else {
                this.seats = 0;
                this.availableSeats = [];
                this.reservedSeats = [];
                this.aviable = 0;
            }
        },
        generateAvailableSeats(seatMap, reservedSeats) {

            const availableSeats = [];
            seatMap.forEach((row) => {
                row.forEach((seat) => {
                    if (seat.label && !reservedSeats.includes(Number(seat.label))) {
                        availableSeats.push(Number(seat.label));
                    }
                });
            });

            return availableSeats;
        },
        isSeatReserved(seat) {
            return this.reservedSeats.includes(Number(seat));
        },
        toggleSeat(seat) {
            if (this.isSeatAvailable(seat)) {
                const seatLabel = Number(seat.label);
                const index = this.selectedSeats.indexOf(seatLabel);
                if (index === -1) {
                    // Si el asiento no está seleccionado, agregarlo
                    this.selectedSeats.push(seatLabel);
                } else {
                    // Si el asiento ya está seleccionado, removerlo
                    this.selectedSeats.splice(index, 1);
                }
                // Forzar la validación del campo después de cambiar selectedSeats
                this.$refs.seatsField.validate();
            }
        },
        calculateTotal() {
            //this.editedItem.total = this.editedItem.price * this.editedItem.quantity;
            const price = Number(this.editedItem.price) || 0;
            const quantity = Number(this.editedItem.quantity) || 0;
            const adults = Number(this.editedItem.adults) || 0;
            const minors = Number(this.editedItem.minors) || 0;

            if (adults === 0 && minors === 0) {
                // Si no hay adultos ni menores, calcular el total por cantidad total
                this.editedItem.total = price * quantity;
            } else {
                // Si hay adultos o menores, calcular el total considerando el 50%
                const discountedTickets = adults + minors;
                this.editedItem.total = price * quantity - discountedTickets * (price * 0.5);
            }
        },
        validateQuantity() {
            const adults = Number(this.editedItem.adults);
            const minors = Number(this.editedItem.minors);
            const quantity = Number(this.editedItem.quantity);
            return adults + minors <= quantity;
        },
        updateDate(val) {
            this.input = val;
            this.editedItem.date = this.dateFormatted;
            this.menu = false;
        },
        async showAdd() {
            this.aviable = '';
            this.data = {};
            this.data.branch_id = Number(this.branch_id);
            try {
                const result = await handleRequest({
                    endpoint: "get-trip-date",
                    method: "POST",
                    data: this.data,
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.trips = result.data?.trips || [];
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.trips = [];
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
            this.selectedSeats = [];
            this.editedIndex = -1;
            this.reservedSeats = [];
        },
        async initialize() {
            try {
                this.loading = true;
                this.data = {};
                this.data.date = format(new Date(), 'yyyy-MM-dd');
                this.data.branch_id = Number(this.branch_id);
                const result = await handleRequest({
                    endpoint: "get-tickets-date",
                    method: "POST",
                    data: this.data
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.tickets = result.data?.tickets || [];
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.tickets = [];
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
        areSeatsDifferent(originalSeats, editedSeats) {
            // Convertir ambos arrays en cadenas de texto para una comparación profunda
            const originalSeatsString = JSON.stringify(originalSeats);
            const editedSeatsString = JSON.stringify(editedSeats);
            // Comparar las cadenas generadas
            return originalSeatsString !== editedSeatsString;
        },
        async save() {
            this.loading = true;
            if (this.editedIndex === -1) {
                this.valid = false;
                const fieldsToUpdate = [
                    "trip_id",
                    "branch_id",
                    "status",
                    "date",
                    "method",
                    "quantity",
                    "price",
                    "total",
                    "seats",
                    "adults",
                    "minors",
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
                if (this.areSeatsDifferent(this.selectedSeats, this.originalItem.seats)) {
                    updatedFields.seats = this.selectedSeats;
                }
                if (Object.keys(updatedFields).length > 0) {
                    updatedFields.date = this.editedItem.date ? this.editedItem.date : new Date();
                    updatedFields.branch_id = this.branch_id;
                    try {
                        const result = await handleRequest({
                            endpoint: "ticket-web",
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
                            this.editedIndex = -1;
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
                        this.editedIndex = -1;
                    }
                }
            } else {
                this.valid = false;
                const fieldsToUpdate = [
                    "trip_id",
                    "branch_id",
                    "status",
                    "date",
                    "method",
                    "quantity",
                    "price",
                    "total",
                    "seats",
                    "adults",
                    "minors",
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

                if (this.areSeatsDifferent(this.originalItem.seats, this.selectedSeats)) {
                    updatedFields.seats = _.cloneDeep(this.selectedSeats);
                }
                if (Object.keys(updatedFields).length > 0) {
                    updatedFields.id = this.editedItem.id;
                    updatedFields.trip_id = this.editedItem.trip_id;
                    try {
                        const result = await handleRequest({
                            endpoint: "ticket",
                            method: "PUT",
                            data: updatedFields,
                        });

                        // Manejo de la respuesta según el resultado
                        if (result.success) {
                            this.showAlert("success", result.message, 3000);
                            this.initialize();
                            this.loading = false;
                        } else {
                            this.editedIndex = -1;
                            this.showAlert("warning", result.message, 3000);
                            this.loading = false;
                        }
                    } catch (error) {
                        this.editedIndex = -1;
                        // Este bloque captura errores inesperados fuera del manejo estándar
                        this.showAlert(
                            "error",
                            "Ocurrió un error inesperado al procesar la solicitud.",
                            3000
                        );
                        this.loading = false;
                    }
                } else {
                    this.editedIndex = -1;
                    this.showAlert("success", "No se realizaron cambios.", 3000);
                    this.loading = false;
                }
            }
            this.close();
        },
        async editItem(item) {
            this.editedIndex = 1;
            this.aviable = '';
            //this.originalItem = Object.assign({}, item);
            //this.editedItem = Object.assign({}, item);
            this.originalItem = _.cloneDeep(item);
            this.editedItem = _.cloneDeep(item);
            this.selectedSeats = item.seats;
            this.data = {};
            this.data.branch_id = this.branch_id;
            this.data.ticket_id = item.id;
            try {
                const result = await handleRequest({
                    endpoint: "get-trip-date",
                    method: "POST",
                    data: this.data,
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.trips = result.data?.trips || [];
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.trips = [];
                }
            } catch (error) {
                this.showAlert("error", "Ocurrió un error inesperado al procesar la solicitud.", 3000);
            } finally {
                this.updateSeats(item.trip_id);
                this.dialog = true;
            }
        },
        deleteItem(item) {
            this.editedIndex = -1;
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
            this.loading = true;
            try {
                let request = {
                    id: this.editedItem.id,
                };
                const result = await handleRequest({
                    endpoint: "ticket-destroy",
                    method: "POST",
                    data: request,
                });

                // Manejo de la respuesta según el resultado
                if (result.success) {
                    this.showAlert("success", result.message, 3000);
                    this.initialize();
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
<style>
.seat-map-preview {
    margin-top: 10px;
}
</style>
