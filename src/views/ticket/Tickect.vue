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
            <v-toolbar :color="paleteColors.primary">
                <v-row align="center">
                    <v-col cols="12" md="8" class="grow ml-4">
                        <span class="text-subtitle-1"><strong>Ticket Vendidos</strong></span>
                    </v-col>
                    <v-col cols="12" md="3" class="text-right">
                        <v-btn class="text-subtitle-1 ml-12" :color="paleteColors.white" variant="tonal" elevation="2"
                            prepend-icon="mdi-plus-circle" @click="showAdd">
                            Vender Ticket
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
                                    <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="branch_id"
                                        v-if="mostrarFila" :items="branches" label="Seleccione una Sucursal"
                                        prepend-inner-icon="mdi-store" item-title="name" item-value="id"
                                        variant="underlined" :rules="selectRules" density="compact">
                                        <template v-slot:item="{ props, item }">
                                            <v-list-item v-bind="props"
                                                :prepend-avatar="`${this.$axios.defaults.baseURL}images/${item.raw.image}`">
                                            </v-list-item>
                                        </template>
                                    </v-autocomplete><!-- @update:model-value="initialize()">-->
                                </v-col>
                                <v-col cols="12" md="2">
                                    <v-btn icon @click="initialize" :color="paleteColors.primary" density="comfortable">
                                        <v-icon>mdi-magnify</v-icon></v-btn>
                                </v-col>
                            </v-row>
                        </v-cols>
                    </v-container>
                </v-row>
                <v-row dense>
                    <v-col cols="12">
                        <v-text-field class="mt-1 mb-1" v-model="search" append-icon="mdi-magnify" label="Buscar"
                            single-line hide-details>
                        </v-text-field>

                        <v-data-table :headers="headers" :search="search" :items="tickets" class="elevation-1"
                            style="max-height: 65vh; overflow-y: auto;" :items-per-page-text="'Elementos por páginas'"
                            no-data-text="No hay datos disponibles" :loading="loading" loading-text="Cargando datos...">
                            <template v-slot:item.actions="{ item }">
                                <v-btn density="comfortable" icon="mdi-pencil" @click="editItem(item)"
                                    :color="paleteColors.primary" variant="tonal" elevation="1"
                                    title="Editar Ticket"></v-btn>
                                <v-btn density="comfortable" icon="mdi-delete" @click="deleteItem(item)"
                                    :color="paleteColors.error" variant="tonal" elevation="1"
                                    title="Eliminar Ticket"></v-btn>
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
                    </v-col>
                </v-row>
            </v-card-text>
        </v-card>
    </v-container>
    <v-dialog v-model="dialog" fullscreen transition="dialog-bottom-transition">
        <v-form ref="form" v-model="valid" enctype="multipart/form-data">
            <v-card style="height: 100vh;">
                <v-toolbar :color="paleteColors.primary">
                    <span class="text-subtitle-2 ml-4">{{ formTitle }}</span>
                </v-toolbar>
                <v-card-text>
                    <v-row style="margin-top: 5px">
                        <!-- Selección de viaje -->
                        <v-col cols="12" md="9">
                            <v-autocomplete :no-data-text="'No hay datos disponibles'" v-model="editedItem.trip_id"
                                :items="trips" label="Rutas" prepend-icon="mdi-road" item-title="name" item-value="id"
                                variant="underlined" :rules="selectRules" density="compact"
                                @update:model-value="updateSeats" :menu-props="{ maxHeight: 400, maxWidth: 600 }" >
                                <template v-slot:item="{ props, item }">
                                    <v-card class="mx-1 my-2" elevation="2">
                                        <v-list-item v-bind="props">
                                            <v-list-item-content>
                                                <v-row align="center" no-gutters>
                                                    <!-- Columna 1: Origen -->
                                                    <v-col cols="12" md="4" class="d-flex align-center">
                                                        <v-avatar>
                                                            <v-img
                                                                :src="`${this.$axios.defaults.baseURL}images/${item.raw.originImage}`"
                                                                max-width="40" />
                                                        </v-avatar>
                                                        <div class="ml-2">
                                                            <div class="text-caption text-grey">
                                                                <v-icon small class="mr-1">mdi-map-marker</v-icon>
                                                                Origen
                                                            </div>
                                                            <v-tooltip location="top">
                                                                <template v-slot:activator="{ props: tooltipProps }">
                                                                    <div v-bind="tooltipProps" class="text-truncate"
                                                                        style="max-width: 100%;">
                                                                        {{ item.raw.origin }}
                                                                    </div>
                                                                </template>
                                                                <span>{{ item.raw.origin }}</span>
                                                                <!-- Texto completo en el tooltip -->
                                                            </v-tooltip>
                                                        </div>
                                                    </v-col>

                                                    <!-- Columna 2: Destino -->
                                                    <v-col cols="12" md="4" class="d-flex align-center">
                                                        <v-avatar>
                                                            <v-img
                                                                :src="`${this.$axios.defaults.baseURL}images/${item.raw.destinationImage}`"
                                                                max-width="40" />
                                                        </v-avatar>
                                                        <div class="ml-2">
                                                            <div class="text-caption text-grey">
                                                                <v-icon small class="mr-1">mdi-map-marker-check</v-icon>
                                                                Destino
                                                            </div>
                                                            <v-tooltip location="top">
                                                                <template v-slot:activator="{ props: tooltipProps }">
                                                                    <div v-bind="tooltipProps" class="text-truncate"
                                                                        style="max-width: 100%;">
                                                                        {{ item.raw.destination }}
                                                                    </div>
                                                                </template>
                                                                <span>{{ item.raw.destination }}</span>
                                                                <!-- Texto completo en el tooltip -->
                                                            </v-tooltip>
                                                        </div>
                                                    </v-col>

                                                    <!-- Columna 3: Horario, Llegada y Vehículo -->
                                                    <v-col cols="12" md="4" class="d-flex align-center">
                                                        <div>
                                                            <div class="text-truncate">
                                                                <v-icon small class="mr-1">mdi-clock-outline</v-icon>
                                                                <strong>Salida:</strong> {{ item.raw.schedule }}
                                                            </div>
                                                            <div class="text-truncate">
                                                                <v-icon small
                                                                    class="mr-1">mdi-clock-check-outline</v-icon>
                                                                <strong>Llegada:</strong> {{ item.raw.arrival }}
                                                            </div>
                                                            <div class="text-truncate">
                                                                <v-icon small class="mr-1">mdi-bus</v-icon>
                                                                <strong>Vehículo:</strong>
                                                                <v-avatar>
                                                                    <v-img
                                                                        :src="`${this.$axios.defaults.baseURL}images/${item.raw.imageVehicle}`"
                                                                        max-width="40" />
                                                                </v-avatar>
                                                                {{ item.raw.plate }}
                                                            </div>
                                                        </div>
                                                    </v-col>
                                                </v-row>
                                            </v-list-item-content>
                                        </v-list-item>
                                    </v-card>
                                </template>
                            </v-autocomplete>
                        </v-col>
                        <v-col cols="12" md="3"></v-col>

                        <!-- Método de pago -->
                        <v-col cols="12" md="2">
                            <v-select v-model="editedItem.method" :items="paymentMethods" label="Método de pago"
                                item-value="value" item-title="text" variant="underlined" density="compact"
                                :rules="[(v) => !!v || 'Seleccione un método de pago']" prepend-icon="mdi-cash">
                                <template v-slot:item="{ props, item }">
                                    <v-list-item v-bind="props">
                                        <template v-slot:prepend>
                                            <v-icon :icon="item.raw.icon"></v-icon> <!-- Ícono de la opción -->
                                        </template>
                                    </v-list-item>
                                </template>
                            </v-select>
                        </v-col>

                        <!-- Fecha -->
                        <v-col cols="12" md="2">
                            <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40"
                                transition="scale-transition" offset-y min-width="190px" disabled="true">
                                <template v-slot:activator="{ props }">
                                    <v-text-field v-bind="props" :modelValue="dateFormatted" variant="underlined"
                                        prepend-icon="mdi-calendar" label="Fecha" density="compact"></v-text-field>
                                </template>
                                <v-locale-provider locale="es">
                                    <v-date-picker header="Calendario" title="Seleccione la fecha"
                                        :color="paleteColors.primary" :modelValue="input"
                                        @update:model-value="updateDate" format="yyyy-MM-dd"
                                        :min="new Date().toISOString().split('T')[0]"></v-date-picker>
                                </v-locale-provider>
                            </v-menu>
                        </v-col>

                        <!-- Precio del pasaje -->
                        <v-col cols="12" md="2">
                            <v-text-field v-model="editedItem.price" label="Precio del pasaje" type="number"
                                variant="underlined" density="compact" prepend-icon="mdi-cash"
                                :rules="[(v) => v > 0 || 'Debe ser un precio válido']"
                                placeholder="Ingrese el precio del pasaje" min="0" step="0.01" readonly></v-text-field>
                        </v-col>

                        <!-- Cantidad de pasajes -->
                        <v-col cols="12" md="2">
                            <v-text-field v-model="editedItem.quantity" label="Cantidad de pasajes" type="number"
                                variant="underlined" density="compact" prepend-icon="mdi-ticket"
                                placeholder="Ingrese la cantidad" min="1" @update:model-value="calculateTotal"
                                :rules="quantityAndPassengerRules" :disabled="!editedItem.trip_id || !aviable"
                                :hint="!editedItem.quantity ? `Asientos disponibles: ${aviable}` : ''"
                                persistent-hint></v-text-field>
                        </v-col>

                        <!-- Selección de asientos -->
                        <v-col cols="12" md="2" v-if="false">
                                        <v-text-field :value="selectedSeats.length > 0 ? selectedSeats.join(', ') : 'Seleccionar Asientos'"
                                            color="primary" dark readonly style="text-transform: none"
                                            :disabled="editedItem.quantity <= 0" prepend-icon="mdi-seat" density="compact"
                                            variant="underlined" :rules="[v => selectedSeats.length > 0 || 'Debe seleccionar al menos un asiento']"></v-text-field>
                                    
                        </v-col>
                    </v-row>
                    <!-- Pasajeros adultos y menores -->
                    <v-row>
                        <v-col cols="12" md="6">
                            <!-- Pasajeros estándar -->
                            <v-card class="pa-4 mb-4">
                                <v-row>
                                    <!-- Campo para pasajeros estándar -->
                                    <v-col cols="12" md="6">
                                        <v-text-field v-model="normal" label="Pasajeros Estandar" type="number"
                                            variant="underlined" density="compact" prepend-icon="mdi-account"
                                            placeholder="Ingrese la cantidad de adultos" min="0"
                                            @update:model-value="onNormalsChange"
                                            :rules="quantityAndPassengerRules" :disabled="isDisabledNormal"></v-text-field>
                                    </v-col>

                                    <!-- Botón para aplicar promoción o autocomplete para seleccionar promoción -->
                                    <v-col cols="12" md="6">
                                        <!-- Mostrar botón "Aplicar Promoción" solo si `normal` tiene valor y no hay promoción seleccionada -->
                                        <v-btn v-if="!showPromotion" @click="showPromotion = showPromotionField" variant="outlined"
                                            prepend-icon="mdi-tag" color="primary" :disabled="isDisabledNormal">
                                            Aplicar Promoción
                                        </v-btn>

                                        <!-- Mostrar autocomplete y botón "Eliminar Promoción" si `showPromotionField` es true -->
                                        <div v-if="this.showPromotion">
                                            <v-autocomplete :no-data-text="'No hay datos disponibles'"
                                                v-model="selectedPromotion" :items="promotions"
                                                label="Seleccionar promoción" item-title="name" item-value="id"
                                                variant="underlined" density="compact" prepend-icon="mdi-tag"
                                                @update:model-value="applyPromotionNormal">
                                                <template v-slot:item="{ props, item }">
                                                    <v-list-item v-bind="props">
                                                        <v-list-item-subtitle>
                                                            <strong>Descuento:</strong> {{ item.raw.percentage }}%
                                                        </v-list-item-subtitle>
                                                        <v-list-item-subtitle>
                                                            <v-tooltip bottom>
                                                                <template v-slot:activator="{ props }">
                                                                    <div class="truncate" v-bind="props"
                                                                        style="max-width: 200px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
                                                                        <strong>Descripción:</strong> {{
                                                                            item.raw.description }}
                                                                    </div>
                                                                </template>
                                                                <span>{{ item.raw.description }}</span>
                                                            </v-tooltip>
                                                        </v-list-item-subtitle>
                                                    </v-list-item>
                                                </template>
                                            </v-autocomplete>
                                            <!-- Botón para eliminar promoción -->
                                            <v-btn v-if="selectedPromotion" @click="removePromotionNormal" variant="text"
                                                color="error" prepend-icon="mdi-close">
                                                Eliminar Promoción
                                            </v-btn>
                                        </div>
                                    </v-col>
                                </v-row>
                            </v-card>

                            <!-- Pasajeros adultos mayores -->
                            <v-card class="pa-4 mb-4">
                                <v-row>
                                    <v-col cols="12" md="6">
                                        <v-text-field v-model="editedItem.adults" label="Pasajeros adultos mayor"
                                            type="number" variant="underlined" density="compact"
                                            prepend-icon="mdi-account-supervisor"
                                            placeholder="Ingrese la cantidad de adultos" min="0"
                                            @update:model-value="onAdultsChange"
                                            :rules="quantityAndPassengerRules"
                                            :disabled="isDisabledAdult"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" md="6">
                                        <v-btn v-if="!showPromotionAdults" @click="showPromotionAdults = showPromotionFieldAdults"
                                            variant="outlined" prepend-icon="mdi-tag" color="primary" :disabled="isDisabledAdult">
                                            Aplicar Promoción
                                        </v-btn>
                                        <div v-if="this.showPromotionAdults">
                                            <v-autocomplete :no-data-text="'No hay datos disponibles'"
                                                v-model="selectedPromotionAdults" :items="promotions"
                                                label="Seleccionar promoción" item-title="name" item-value="id"
                                                variant="underlined" density="compact" prepend-icon="mdi-tag"
                                                @update:model-value="applyPromotionAdults">
                                                <template v-slot:item="{ props, item }">
                                                    <v-list-item v-bind="props">
                                                        <v-list-item-subtitle>
                                                            <strong>Descuento:</strong> {{ item.raw.percentage }}%
                                                        </v-list-item-subtitle>
                                                        <v-list-item-subtitle>
                                                            <v-tooltip bottom>
                                                                <template v-slot:activator="{ props }">
                                                                    <div class="truncate" v-bind="props"
                                                                        style="max-width: 200px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
                                                                        <strong>Descripción:</strong> {{
                                                                            item.raw.description }}
                                                                    </div>
                                                                </template>
                                                                <span>{{ item.raw.description }}</span>
                                                            </v-tooltip>
                                                        </v-list-item-subtitle>
                                                    </v-list-item>
                                                </template>
                                            </v-autocomplete>
                                            <v-btn v-if="selectedPromotionAdults" @click="removePromotionAdult" variant="text" color="error"
                                                prepend-icon="mdi-close">
                                                Eliminar Promoción
                                            </v-btn>
                                        </div>
                                    </v-col>
                                </v-row>
                            </v-card>

                            <!-- Pasajeros menores de edad -->
                            <v-card class="pa-4 mb-4">
                                <v-row>
                                    <v-col cols="12" md="6">
                                        <v-text-field v-model="editedItem.minors" label="Pasajeros menores de edad"
                                            type="number" variant="underlined" density="compact"
                                            prepend-icon="mdi-account-child"
                                            placeholder="Ingrese la cantidad de menores" min="0"
                                            @update:model-value="onMinorsChange"
                                            :rules="quantityAndPassengerRules" :disabled="isDisabledMinor"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" md="6">
                                        <v-btn v-if="!showPromotionMinors" @click="showPromotionMinors = showPromotionFieldMinors"
                                            variant="outlined" prepend-icon="mdi-tag" color="primary" :disabled="isDisabledMinor">
                                            Aplicar Promoción
                                        </v-btn>
                                        <div v-if="this.showPromotionMinors">
                                            <v-autocomplete :no-data-text="'No hay datos disponibles'"
                                                v-model="selectedPromotionMinors" :items="promotions"
                                                label="Seleccionar promoción" item-title="name" item-value="id"
                                                variant="underlined" density="compact" prepend-icon="mdi-tag"
                                                @update:model-value="applyPromotionMinors">
                                                <template v-slot:item="{ props, item }">
                                                    <v-list-item v-bind="props">
                                                        <v-list-item-subtitle>
                                                            <strong>Descuento:</strong> {{ item.raw.percentage }}%
                                                        </v-list-item-subtitle>
                                                        <v-list-item-subtitle>
                                                            <v-tooltip bottom>
                                                                <template v-slot:activator="{ props }">
                                                                    <div class="truncate" v-bind="props"
                                                                        style="max-width: 200px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">
                                                                        <strong>Descripción:</strong> {{
                                                                            item.raw.description }}
                                                                    </div>
                                                                </template>
                                                                <span>{{ item.raw.description }}</span>
                                                            </v-tooltip>
                                                        </v-list-item-subtitle>
                                                    </v-list-item>
                                                </template>
                                            </v-autocomplete>
                                            <v-btn v-if="selectedPromotionMinors" @click="removePromotionMinor" variant="text" color="error"
                                                prepend-icon="mdi-close">
                                                Eliminar Promoción
                                            </v-btn>
                                        </div>
                                    </v-col>
                                </v-row>
                            </v-card>

                            <!-- Total a pagar -->
                            <v-card class="pa-4">
                                <v-row>
                                    <v-col cols="12" md="6">
                                        <v-text-field v-model="editedItem.total" label="Total a pagar" type="number"
                                            variant="underlined" density="compact" prepend-icon="mdi-cash"
                                            readonly></v-text-field>
                                    </v-col>
                                </v-row>
                            </v-card>
                        </v-col>
                        <v-col cols="12" md="6">
                            <!-- Mapa de asientos visible -->
                            <v-card style="max-width: 50%;" v-if="aviable">
                                <v-toolbar :color="paleteColors.primary">
                                    <span class="text-subtitle-2 ml-4">Seleccione los asientos</span>
                                </v-toolbar>
                                <v-card-text>
                                    <v-row>
                                        <!-- Mostrar asientos en filas de 2 -->
                                        <v-col cols="12" class="d-flex align-center justify-center">
                                            <div class="seat-map-preview"
                                                style="display: flex; flex-direction: column;">
                                                <div v-for="(row, rowIndex) in seatMap" :key="rowIndex" class="seat-row"
                                                    style="display: flex; flex-direction: row;">
                                                    <template v-for="(seat, seatIndex) in row" :key="seatIndex">
                                                        <!--<v-btn v-if="seat.type" :color="getSeatColor(seat)"
                                                            class="seat-button-preview ma-1"
                                                            :disabled="!isSeatAvailable(seat)" @click="toggleSeat(seat)"
                                                            style="min-width: 30px; min-height: 30px; font-size: 0.8rem; font-weight: bold;">
                                                            <v-icon v-if="seat.type === 'seat'">mdi-seat</v-icon>
                                                            <span v-if="seat.type === 'aisle'">
                                                                <v-icon>mdi-arrow-down</v-icon> P
                                                            </span>
                                                            {{ seat.type === 'seat' ? seat.label : '' }}
                                                        </v-btn>-->
                                                        <div v-if="seat.type" 
                                                            :class="['seat-icon-preview', 'ma-1', { 'disabled': !isSeatAvailable(seat) }]" 
                                                            :style="{ color: getSeatColor(seat) }" 
                                                            @click="toggleSeat(seat)" 
                                                            style="cursor: pointer; font-weight: bold; position: relative;">
                                                            
                                                            <!-- Mostrar ícono de asiento -->
                                                            <v-icon v-if="seat.type === 'seat'" size="x-large" class="seat-icon">
                                                            mdi-seat
                                                            </v-icon>
                                                            
                                                            <!-- Mostrar ícono de pasillo -->
                                                            <v-icon v-if="seat.type === 'aisle'" size="x-large" class="aisle-icon">
                                                            mdi-arrow-split-vertical
                                                            </v-icon>

                                                            <!-- Superponer el número del asiento -->
                                                            <span v-if="seat.type === 'seat'" class="seat-label">{{ seat.label }}</span>

                                                            <!-- Superponer la "P" del pasillo -->
                                                            <span v-if="seat.type === 'aisle'" class="aisle-label">P</span>
                                                        </div>
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
                            </v-card>
                        </v-col>
                    </v-row>
                </v-card-text>
                <v-divider></v-divider>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn :color="paleteColors.gris" variant="flat" @click="close">Cancelar</v-btn>
                    <v-btn :color="paleteColors.primary" variant="flat" @click="save"
                        :disabled="!valid || Number(selectedSeats.length) !== Number(editedItem.quantity)"
                        :loading="loading">Aceptar</v-btn>
                </v-card-actions>
            </v-card>
        </v-form>
    </v-dialog>
    <v-dialog v-model="dialogDelete" max-width="500px">
        <v-card>
            <v-toolbar :color="paleteColors.error">
                <span class="text-subtitle-2 ml-4"> Eliminar un Ticket</span>
            </v-toolbar>

            <v-card-text class="mt-2 mb-2"> ¿Desea eliminar el ticket seleccionado?</v-card-text>
            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn :color="paleteColors.gris" variant="flat" @click="closeDelete"> Cancelar </v-btn>
                <v-btn :color="paleteColors.error" variant="flat" @click="deleteItemConfirm" :loading="loading">
                    Aceptar
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import _ from 'lodash';
import { paleteColors } from "@/assets/colors";
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
        dialogDelete: false,
        branch_id: '',
        trips: [],
        routes: [],
        vehicles: [],
        workers: [],
        tickets: [],
        promotions: [],
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
            promotions: [],
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
            promotions: [],
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
            promotions: [],
        },
        paymentMethods: [
            { text: "Efectivo", value: "Efectivo", icon: "mdi-cash" },
            { text: "Débito", value: "Debito", icon: "mdi-credit-card-outline" },
            { text: "Crédito", value: "Credito", icon: "mdi-credit-card-multiple-outline" },
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
        appliedPromotions: [],
        selectedPromotionAdults: null,
        selectedPromotionMinors: null,
        selectedPromotion: null,
        showPromotion: false,
        showPromotionAdults: false,
        showPromotionMinors: false,
        normal: '',
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
            return this.editedIndex === -1 ? "Venta de Ticket" : "Editar Ticket";
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
                    return "La suma de pasajeros, adultos y menores no puede ser mayor que la cantidad de pasajes.";
                },
            ];
        },
        showPromotionFieldAdults() {
            return this.editedItem.adults > 0 && this.promotions.length > 0;
        },
        showPromotionFieldMinors() {
            return this.editedItem.minors > 0 && this.promotions.length > 0;
        },
        showPromotionField() {
            return this.normal > 0 && this.promotions.length > 0;
        },
        isDisabledNormal() {
            return (Number(this.editedItem.adults) + Number(this.editedItem.minors)) >= Number(this.editedItem.quantity);
        },
        isDisabledAdult() {
            return (Number(this.normal) + Number(this.editedItem.minors)) >= Number(this.editedItem.quantity);
        },
        isDisabledMinor() {
            return (Number(this.normal) + Number(this.editedItem.adults)) >= Number(this.editedItem.quantity);
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
            this.normal = Math.min(this.normal, newValue);
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
        applyPromotionAdults(promotionId) {
            // Buscar la promoción seleccionada
            const selectedPromotion = this.promotions.find((promo) => promo.id === promotionId);

            if (selectedPromotion) {
                // Calcular el nuevo descuento
                const newDiscountAmount = this.editedItem.adults * [(this.editedItem.price * selectedPromotion.percentage) / 100];

                // Buscar si ya existe un registro de tipo "adults"
                const existingPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "adults"
                );

                if (existingPromotionIndex !== -1) {
                    // Obtener la promoción existente
                    const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                    // Calcular la diferencia entre el descuento anterior y el nuevo
                    const discountDifference = newDiscountAmount - existingPromotion.discountedPrice;

                    // Actualizar TODOS los campos del registro existente
                    this.editedItem.promotions[existingPromotionIndex] = {
                        id: selectedPromotion.id, // Actualizar el ID de la promoción
                        percentage: selectedPromotion.percentage, // Actualizar el porcentaje
                        originalPrice: this.editedItem.price, // Actualizar el precio original
                        discountedPrice: newDiscountAmount, // Actualizar el descuento calculado
                        type: "adults", // Mantener el tipo
                    };

                    // Actualizar el total sumando o restando la diferencia
                    this.editedItem.total -= discountDifference;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción actualizada adults:", selectedPromotion);
                    console.log("Diferencia de descuento adults:", discountDifference);
                    console.log("Total actualizado adults:", this.editedItem.total);
                } else {
                    // Si no existe un registro, agregar uno nuevo
                    this.editedItem.promotions.push({
                        id: selectedPromotion.id,
                        percentage: selectedPromotion.percentage,
                        originalPrice: this.editedItem.price,
                        discountedPrice: newDiscountAmount,
                        type: "adults",
                    });

                    // Actualizar el total restando el nuevo descuento
                    this.editedItem.total -= newDiscountAmount;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción aplicada adults:", selectedPromotion);
                    console.log("Total después del descuento adults:", this.editedItem.total);
                }

                console.log("Promociones aplicadas adults:", this.editedItem.promotions);
            }
        },
        removePromotionAdult() {
            // Buscar la promoción de tipo "normal"
            const existingPromotionIndex = this.editedItem.promotions.findIndex(
                (promo) => promo.type === "adults"
            );

            if (existingPromotionIndex !== -1) {
                // Obtener la promoción existente
                const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                // Sumar el valor del descuento al total
                this.editedItem.total += existingPromotion.discountedPrice;

                // Eliminar la promoción de la lista
                this.editedItem.promotions.splice(existingPromotionIndex, 1);

                // Reiniciar la promoción seleccionada
                this.selectedPromotionAdults = null;
                this.showPromotionAdults = false;

                // Mostrar un mensaje de éxito (opcional)
                console.log("Promoción eliminada:", existingPromotion);
                console.log("Total restaurado:", this.editedItem.total);
            }
        },
        onAdultsChange(newValue) {
            if (newValue == 0) {
                // Buscar y eliminar la promoción de tipo "adults"
                const adultPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "adults"
                );

                if (adultPromotionIndex !== -1) {
                    // Obtener la promoción eliminada
                    const removedPromotion = this.editedItem.promotions[adultPromotionIndex];

                    // Eliminar la promoción del array
                    this.editedItem.promotions.splice(adultPromotionIndex, 1);

                    // Actualizar el total sumando el descuento que se había aplicado
                    this.editedItem.total += removedPromotion.discountedPrice;
                    this.selectedPromotionAdults = null;
                    this.showPromotionAdults = false;
                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción eliminada adults:", removedPromotion);
                    console.log("Total actualizado adults:", this.total);
                    console.log("Promociones aplicadas adults:", this.editedItem.promotions);
                }
            } else {
                if (this.selectedPromotionAdults) {
                    this.applyPromotionAdults(this.selectedPromotionAdults);
                }
            }
        },
        applyPromotionMinors(promotionId) {
            // Buscar la promoción seleccionada
            const selectedPromotion = this.promotions.find((promo) => promo.id === promotionId);

            if (selectedPromotion) {
                // Calcular el nuevo descuento
                const newDiscountAmount = this.editedItem.minors * [(this.editedItem.price * selectedPromotion.percentage) / 100];

                // Buscar si ya existe un registro de tipo "adults"
                const existingPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "minors"
                );

                if (existingPromotionIndex !== -1) {
                    // Obtener la promoción existente
                    const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                    // Calcular la diferencia entre el descuento anterior y el nuevo
                    const discountDifference = newDiscountAmount - existingPromotion.discountedPrice;

                    // Actualizar TODOS los campos del registro existente
                    this.editedItem.promotions[existingPromotionIndex] = {
                        id: selectedPromotion.id, // Actualizar el ID de la promoción
                        percentage: selectedPromotion.percentage, // Actualizar el porcentaje
                        originalPrice: this.editedItem.price, // Actualizar el precio original
                        discountedPrice: newDiscountAmount, // Actualizar el descuento calculado
                        type: "minors", // Mantener el tipo
                    };

                    // Actualizar el total sumando o restando la diferencia
                    this.editedItem.total -= discountDifference;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción actualizada minors:", selectedPromotion);
                    console.log("Diferencia de descuento minors:", discountDifference);
                    console.log("Total actualizado minors:", this.editedItem.total);
                } else {
                    // Si no existe un registro, agregar uno nuevo
                    this.editedItem.promotions.push({
                        id: selectedPromotion.id,
                        percentage: selectedPromotion.percentage,
                        originalPrice: this.editedItem.price,
                        discountedPrice: newDiscountAmount,
                        type: "minors",
                    });

                    // Actualizar el total restando el nuevo descuento
                    this.editedItem.total -= newDiscountAmount;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción aplicada minors:", selectedPromotion);
                    console.log("Total después del descuento minors:", this.editedItem.total);
                }

                console.log("Promociones aplicadas minors:", this.editedItem.promotions);
            }
        },
        removePromotionMinor() {
            // Buscar la promoción de tipo "normal"
            const existingPromotionIndex = this.editedItem.promotions.findIndex(
                (promo) => promo.type === "minors"
            );

            if (existingPromotionIndex !== -1) {
                // Obtener la promoción existente
                const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                // Sumar el valor del descuento al total
                this.editedItem.total += existingPromotion.discountedPrice;

                // Eliminar la promoción de la lista
                this.editedItem.promotions.splice(existingPromotionIndex, 1);

                // Reiniciar la promoción seleccionada
                this.selectedPromotionMinors = null;
                this.showPromotionMinors = false;

                // Mostrar un mensaje de éxito (opcional)
                console.log("Promoción eliminada:", existingPromotion);
                console.log("Total restaurado:", this.editedItem.total);
            }
        },
        onMinorsChange(newValue) {
            if (newValue == 0) {
                // Buscar y eliminar la promoción de tipo "adults"
                const minorPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "minors"
                );

                if (minorPromotionIndex !== -1) {
                    // Obtener la promoción eliminada
                    const removedPromotion = this.editedItem.promotions[minorPromotionIndex];

                    // Eliminar la promoción del array
                    this.editedItem.promotions.splice(minorPromotionIndex, 1);

                    // Actualizar el total sumando el descuento que se había aplicado
                    this.editedItem.total += removedPromotion.discountedPrice;
                    this.selectedPromotionMinors = null;
                    this.showPromotionMinors = false;
                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción eliminada minors:", removedPromotion);
                    console.log("Total actualizado minors:", this.total);
                    console.log("Promociones aplicadas minors:", this.editedItem.promotions);
                }
            } else {
                if (this.selectedPromotionMinors) {
                    this.applyPromotionMinors(this.selectedPromotionMinors);
                }
            }
        },
        applyPromotionNormal(promotionId) {
            // Buscar la promoción seleccionada
            const selectedPromotion = this.promotions.find((promo) => promo.id === promotionId);

            if (selectedPromotion) {
                // Calcular el nuevo descuento
                const newDiscountAmount = this.normal * [(this.editedItem.price * selectedPromotion.percentage) / 100];

                // Buscar si ya existe un registro de tipo "adults"
                const existingPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "normal"
                );

                if (existingPromotionIndex !== -1) {
                    // Obtener la promoción existente
                    const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                    // Calcular la diferencia entre el descuento anterior y el nuevo
                    const discountDifference = newDiscountAmount - existingPromotion.discountedPrice;

                    // Actualizar TODOS los campos del registro existente
                    this.editedItem.promotions[existingPromotionIndex] = {
                        id: selectedPromotion.id, // Actualizar el ID de la promoción
                        percentage: selectedPromotion.percentage, // Actualizar el porcentaje
                        originalPrice: this.editedItem.price, // Actualizar el precio original
                        discountedPrice: newDiscountAmount, // Actualizar el descuento calculado
                        type: "normal", // Mantener el tipo
                    };

                    // Actualizar el total sumando o restando la diferencia
                    this.editedItem.total -= discountDifference;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción actualizada:", selectedPromotion);
                    console.log("Diferencia de descuento:", discountDifference);
                    console.log("Total actualizado:", this.editedItem.total);
                } else {
                    // Si no existe un registro, agregar uno nuevo
                    this.editedItem.promotions.push({
                        id: selectedPromotion.id,
                        percentage: selectedPromotion.percentage,
                        originalPrice: this.editedItem.price,
                        discountedPrice: newDiscountAmount,
                        type: "normal",
                    });

                    // Actualizar el total restando el nuevo descuento
                    this.editedItem.total -= newDiscountAmount;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción aplicada:", selectedPromotion);
                    console.log("Total después del descuento:", this.editedItem.total);
                }

                console.log("Promociones aplicadas:", this.editedItem.promotions);
            }
        },
        removePromotionNormal() {
            // Buscar la promoción de tipo "normal"
            const existingPromotionIndex = this.editedItem.promotions.findIndex(
                (promo) => promo.type === "normal"
            );

            if (existingPromotionIndex !== -1) {
                // Obtener la promoción existente
                const existingPromotion = this.editedItem.promotions[existingPromotionIndex];

                // Sumar el valor del descuento al total
                this.editedItem.total += existingPromotion.discountedPrice;

                // Eliminar la promoción de la lista
                this.editedItem.promotions.splice(existingPromotionIndex, 1);

                // Reiniciar la promoción seleccionada
                this.selectedPromotion = null;
                this.showPromotion = false;

                // Mostrar un mensaje de éxito (opcional)
                console.log("Promoción eliminada:", existingPromotion);
                console.log("Total restaurado:", this.editedItem.total);
            }
        },
        onNormalsChange(newValue) {
            if (newValue == 0) {
                // Buscar y eliminar la promoción de tipo "adults"
                const minorPromotionIndex = this.editedItem.promotions.findIndex(
                    (promo) => promo.type === "normal"
                );

                if (minorPromotionIndex !== -1) {
                    // Obtener la promoción eliminada
                    const removedPromotion = this.editedItem.promotions[minorPromotionIndex];

                    // Eliminar la promoción del array
                    this.editedItem.promotions.splice(minorPromotionIndex, 1);

                    // Actualizar el total sumando el descuento que se había aplicado
                    this.editedItem.total += removedPromotion.discountedPrice;
                    this.selectedPromotion = null;
                    this.showPromotion = false;

                    // Mostrar un mensaje de éxito (opcional)
                    console.log("Promoción eliminada:", removedPromotion);
                    console.log("Promociones aplicadas:", this.editedItem.promotions);
                }
            } else {
                if (this.selectedPromotion) {
                    this.applyPromotionNormal(this.selectedPromotion);
                }
            }
        },
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
                    this.mostrarFila = false;
                    // Si no hay datos, asignamos un array vacío
                    this.branches = [];
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
        getSeatColor(seat) {
            if (this.isSeatReserved(seat.label)) {
                return this.paleteColors.error; // Asiento reservado
            } else if (this.selectedSeats.includes(Number(seat.label))) {
                return this.paleteColors.primary; // Asiento seleccionado
            } else if (seat.type === 'aisle') {
                return this.paleteColors.gris; // Asiento seleccionado
            }
            else {
                return this.paleteColors.green; // Asiento disponible
            }
        },
        isSeatAvailable(seat) {
            return seat.label && !this.isSeatReserved(seat.label);
        },
        updateSeats(tripId) {
            this.seats = 0;
                this.availableSeats = [];
                this.reservedSeats = [];
                this.aviable = 0;
                this.editedItem.quantity = '';
            this.selectedSeats = [];
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
                //this.$refs.seatsField.validate();
            }
        },
        calculateTotal() {
            this.editedItem.adults = 0;
            this.editedItem.minors = 0;
            this.normal = 0;
            this.selectedPromotionAdults= null;
            this.selectedPromotionMinors= null;
            this.selectedPromotion= null;
            this.showPromotion= false;
            this.showPromotionAdults= false;
            this.showPromotionMinors= false;
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
            const normal = Number(this.normal);
            const quantity = Number(this.editedItem.quantity);
            return adults + minors + normal <= quantity;
        },
        updateDate(val) {
            this.input = val;
            this.editedItem.date = this.dateFormatted;
            this.menu = false;
        },
        async showAdd() {
            this.aviable = '';
            this.normal = '';
            this.selectedPromotion = '';
            this.data = {};
            this.data.branch_id = Number(this.branch_id);
            const today = new Date();
            const formattedDate = today.toISOString().split('T')[0]; // Formato: YYYY-MM-DD
            //this.data.date = formattedDate;
            try {
                const result = await handleRequest({
                    endpoint: "get-trip-date",
                    method: "POST",
                    data: this.data,
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.trips = result.data?.trips || [];
                    this.promotions = result.data?.promotions || [];
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.trips = [];
                    this.promotions = [];
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
                const today = new Date();
                const formattedDate = today.toISOString().split('T')[0]; // Formato: YYYY-MM-DD
                this.data.date = formattedDate;
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
                    "promotions"
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
                    "promotions"
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

            // Inicializar las variables de promoción
            this.selectedPromotion = null;
            this.selectedPromotionAdults = null;
            this.selectedPromotionMinors = null;

            // Buscar en el array de promociones
            if (this.editedItem.promotions && this.editedItem.promotions.length > 0) {
                this.editedItem.promotions.forEach((promotion) => {
                    if (promotion.type === "normal") {
                        this.selectedPromotion = promotion.id; // Asignar el ID de la promoción normal
                        this.normal = item.quantity - item.adults - item.minors; // Asignar la cantidad (si existe)
                    } else if (promotion.type === "adults") {
                        this.selectedPromotionAdults = promotion.id; // Asignar el ID de la promoción para adultos
                    } else if (promotion.type === "minors") {
                        this.selectedPromotionMinors = promotion.id; // Asignar el ID de la promoción para menores
                    }
                });
            }
            const today = new Date();
            const formattedDate = today.toISOString().split('T')[0]; // Formato: YYYY-MM-DD
            //this.data.date = formattedDate;
            try {
                const result = await handleRequest({
                    endpoint: "get-trip-date",
                    method: "POST",
                    data: this.data,
                });

                if (result.success) {
                    // Si la solicitud es exitosa, asignamos las sucursales
                    this.trips = result.data?.trips || [];
                    this.promotions = result.data?.promotions || [];
                } else {
                    // Si no hay datos, asignamos un array vacío
                    this.trips = [];
                    this.promotions = [];
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
<style scoped>
.seat-icon-preview {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  position: relative; /* Necesario para posicionar los elementos hijos de forma absoluta */
}

.disabled {
  opacity: 0.7;
  pointer-events: none;
}

/* Estilos para el ícono de asiento */
.seat-icon {
  font-size: 3rem; /* Tamaño del ícono */
}

/* Estilos para el ícono de pasillo */
.aisle-icon {
  font-size: 3rem; /* Tamaño del ícono */
}

/* Estilos para el número del asiento */
.seat-label {
  position: absolute;
  top: 30%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1rem; /* Tamaño del número */
  font-weight: bold;
  color: black; /* Color del texto */
}

/* Estilos para la "P" del pasillo */
.aisle-label {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1rem; /* Tamaño de la "P" */
  font-weight: bold;
  color: black; /* Color del texto */
}
</style>
