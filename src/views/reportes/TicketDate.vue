<template>
    <v-container style="min-width: 100%; min-height: 100%;">
        <v-row>
            <v-toolbar color="#1976D2">
                <v-row align="center">
                    <v-col cols="12" md="8" class="grow ml-4">
                        <span class="text-subtitle-1"><strong>Ventas Diarías</strong></span>
                    </v-col>
                    <v-col cols="12" md="3" class="text-right">
                        <v-btn class="text-subtitle-1 ml-12" color="white" variant="tonal" elevation="2"
                            prepend-icon="mdi-file-excel-box" @click="exportToExcel">
                            Exportar a Excel
                        </v-btn>
                    </v-col>
                </v-row>
            </v-toolbar>
        </v-row>
        <v-row>
            <v-col cols="12" md="3">
                <v-menu v-model="menu" :close-on-content-click="false" :nudge-right="40" transition="scale-transition"
                    offset-y min-width="290px">
                    <template v-slot:activator="{ props }">
                        <v-text-field v-bind="props" :modelValue="dateFormatted" variant="underlined"
                            prepend-icon="mdi-calendar" label="Fecha de inicio" density="compact"></v-text-field>
                    </template>
                    <v-locale-provider locale="es">
                        <v-date-picker header="Calendario" title="Seleccione la fecha" color="#1976D2"
                            :modelValue="input" @update:model-value="updateDate" format="yyyy-MM-dd"></v-date-picker>
                    </v-locale-provider>
                </v-menu>
            </v-col>
            <v-col cols="12" md="3">
                <v-menu v-model="menu2" :close-on-content-click="false" :nudge-right="40" transition="scale-transition"
                    offset-y min-width="290px">
                    <template v-slot:activator="{ props }">
                        <v-text-field v-bind="props" :modelValue="dateFormatted1" variant="underlined"
                            prepend-icon="mdi-calendar" label="Fecha Terminación" density="compact"></v-text-field>
                    </template>
                    <v-locale-provider locale="es">
                        <v-date-picker header="Calendario" title="Seleccione la fecha" color="#1976D2"
                            :modelValue="input2" format="yyyy-MM-dd" :min="dateFormatted"
                            @update:model-value="updateDate1"></v-date-picker><!--@update:model-value="updateDate2"-->
                    </v-locale-provider>
                </v-menu>
            </v-col>
            <v-col cols="12" md="3">
                <v-btn icon @click="initialize" color="#1976D2">
                    <v-icon>mdi-magnify</v-icon></v-btn>
            </v-col>
        </v-row>
        <v-row class="mx-auto" max-width="400">
            <!-- Contenido del reporte -->
            <v-card ref="reportContent" class="mx-auto" max-width="500" style="max-height: 68vh; overflow-y: auto;">
                <v-card-text>
                    <v-col cols="12" class="pa-0"> <!-- Elimina el padding en la columna principal -->
                        <v-row class="ma-0"> <!-- Elimina el margin en la fila -->
                            <v-col cols="12" class="text-h6 text-center pa-1"> <!-- Ajusta el padding -->
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
                            <v-col cols="12" v-for="(total, index) in this.response.totalesPorMetodo" :key="index"
                                class="pa-1">
                                {{ total.metodo }}: {{ (Number(total.cantidad)) }}
                            </v-col>
                            <v-col cols="12" class="pa-1">
                                <strong>TOTALES:</strong>
                            </v-col>
                            <v-col cols="12" v-for="(total, index) in this.response.totalesPorMetodo" :key="index"
                                class="pa-1">
                                <strong>{{ total.metodo }}:</strong> ${{ total.total }}
                            </v-col>
                            <v-col cols="12" class="font-weight-bold pa-1">
                                TOTAL: ${{ (Number(this.response.totales)) }}
                            </v-col>
                        </v-row>
                    </v-col>
                </v-card-text>
            </v-card>
        </v-row>
    </v-container>
</template>

<script>
import LocalStorageService from "@/LocalStorageService";
import { handleRequest } from "@/utils/api"; // Ruta al archivo
import { format } from 'date-fns';
import * as XLSX from 'xlsx';
export default {
    data: () => ({
        reporteData: {
            nombre: "Empresa XYZ",
            fecha: "2023-10-01",
            pasajesEmitidos: 150,
            reimpresiones: 10,
            totalesPorMetodo: [
                { metodo: "Efectivo", total: 5000 },
                { metodo: "Tarjeta", total: 3000 },
                { metodo: "Transferencia", total: 2000 }
            ],
            totales: 10000
        },
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
        type: 'Sucursal',
        branch_id: '',
        company_id: '',
        role: '',
        response: [],
        data: {},
        menu: false,
        menu2: false,
        input: null,
        input2: null,
        date: null,
        endDate: null,
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
        this.role = JSON.parse(LocalStorageService.getItem('role'));
        this.company_id = LocalStorageService.getItem('business_id');
        if (this.role === 'Administrador') {
            this.showBranches();
            this.type = "Company";
        } else {
            this.type = "Sucursal"
            this.branch_id = LocalStorageService.getItem('branch_id');
        }

    },
    methods: {
        formatNumber(value) {
            // Si el valor es menor que 1000, devuelve el valor original con dos decimales
            if (value < 1000) {
                return (Math.round((value + Number.EPSILON) * 100) / 100).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
            }

            // Primero, redondea el valor a dos decimales
            value = Math.round((value + Number.EPSILON) * 100) / 100;

            // Convierte el valor a cadena con formato de número local (en-US)
            let formattedValue = value.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });

            return formattedValue;
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
            try {
                this.loading = true;
                this.data = {};
                this.data.id = this.type === "Company" ? Number(this.company_id) : Number(this.branch_id);
                this.data.type = this.type;
                // Formatear las fechas
                const formattedDate = this.date ? format(new Date(this.date), 'yyyy-MM-dd') : format(new Date(), 'yyyy-MM-dd');
                const formattedEndDate = this.endDate ? format(new Date(this.endDate), 'yyyy-MM-dd') : format(new Date(), 'yyyy-MM-dd');

                // Comparar las fechas
                if (formattedDate === formattedEndDate) {
                    this.data.date = formattedDate; // Solo enviar una fecha si son iguales
                } else {
                    this.data.date = formattedDate;
                    this.data.endDate = formattedEndDate;
                }
                const result = await handleRequest({
                    endpoint: "ticket-sold-date",
                    method: "POST",
                    data: this.data
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
            //try {
                let rows = [];

                rows.push([this.response.nombre]);
                rows.push([]);
                rows.push(["FECHA:", this.response.fecha]);
                rows.push([]);
                rows.push(["ENISIÓN DE PASAJES"]);
                rows.push([]);
                rows.push(["Pasajes emitidos:", this.response.pasajesEmitidos]);
                rows.push(["Reimpresiones:", this.response.reimpresiones]);
                rows.push([]);
                // 4. Agregar filas con los totales por método de pago  
                this.response.totalesPorMetodo.forEach((item) => {
                    rows.push([item.metodo, `$${item.cantidad}`]);
                });

                rows.push([]); // Fila vacía para separar
                // 3. Agregar encabezados para los totales por método de pago
                rows.push(["TOTALES"]);
                rows.push([]);
                // 4. Agregar filas con los totales por método de pago
                this.response.totalesPorMetodo.forEach((item) => {
                    rows.push([item.metodo, `$${item.total}`]);
                });

                rows.push([]); // Fila vacía para separar

                rows.push(["TOTAL:", `$${this.response.totales}`]);

                const ws = XLSX.utils.aoa_to_sheet(rows);
                const wb = XLSX.utils.book_new();
                XLSX.utils.book_append_sheet(wb, ws, "Reporte");

                XLSX.writeFile(wb, `reporte_ventas_${new Date().toLocaleDateString().replace(/\//g, '-')}.xlsx`);
            //} catch (error) {
                //this.showAlert("error", "Ocurrió un error al exportar el Excel.", 3000);
            //}
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
/* Estilos personalizados */
.v-card {
    border-radius: 10px;
}

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
</style>