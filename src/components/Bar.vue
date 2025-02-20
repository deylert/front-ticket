<template>
   <v-container style="height: 300px;" fluid> <!-- Ajusta la altura aquí -->
    <Bar :options="chartOptions" :data="chartData" />
    </v-container>
  </template>
  <script>
  import { Bar } from "vue-chartjs";
  import {
    Chart as ChartJS,
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale,
  } from "chart.js";
  
  ChartJS.register(
    Title,
    Tooltip,
    Legend,
    BarElement,
    CategoryScale,
    LinearScale
  );
  
  export default {
    name: "BarChart",
    props: {
    dataArray: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      chartOptions: {
        responsive: true,
        maintainAspectRatio: false,
        scales: {
          x: {
            grid: {
              display: false, // Oculta la cuadrícula del eje X
            },
            barThickness: 30, // Ancho fijo de las barras en píxeles
          },
          y: {
            grid: {
              display: true, // Muestra la cuadrícula del eje Y
            },
          },
        },
      },
    };
  },
  computed: {
    chartData() {
      return {
        labels: [
          "Enero",
          "Febrero",
          "Marzo",
          "Abril",
          "Mayo",
          "Junio",
          "Julio",
          "Agosto",
          "Septiembre",
          "Octubre",
          "Noviembre",
          "Diciembre",
        ],
        datasets: [
          {
            label: "Ingresado",
            backgroundColor: "#1976D2",
            data: this.dataArray, // Usamos el array pasado como prop
          },
        ],
      };
    },
  },
    components: {
      Bar,
    },
  };
  </script>
  <style scoped></style>