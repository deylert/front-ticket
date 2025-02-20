 <!--<template>
      <container> Ajusta la altura aquí 
    <Doughnut :data="chartData" :options="chartOptions" />
  </container>
 </template>
 <script>
 import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'
 import { Doughnut } from 'vue-chartjs'
 
 ChartJS.register(ArcElement, Tooltip, Legend)
 
 export default {
   name: 'App',
 
   data: () => ({
 
       chartData: {
         labels: ['Ingresos', 'Pérdidas', 'Beneficios', 'Gastos'],
         datasets: [
     {
       backgroundColor: ['#41B883', 'orange', 'blue', '#DD1B16'],
       data: [278, 120, 320, 45]
     }
   ]
       },
       chartOptions: {
         responsive: true
       }
   }),
   components: {
     Doughnut,
   },
 };
 </script>
 <style scoped>
 
 </style>-->
 <template>
  <v-container>
    <v-row v-for="(row, rowIndex) in seats" :key="rowIndex">
      <v-col v-for="(seat, colIndex) in row" :key="colIndex">
        <v-btn
          :color="selectedSeats.includes(seat) ? 'primary' : 'grey'"
          @click="toggleSeat(seat)"
        >
          {{ seat }}
        </v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      seats: [
        [1, 2, 3, 4],
        [5, 6, 7, 8],
        [9, 10, 11, 12],
        [13, 14, 15, 16]
      ],
      selectedSeats: []
    };
  },
  methods: {
    toggleSeat(seat) {
      const index = this.selectedSeats.indexOf(seat);
      if (index === -1) {
        this.selectedSeats.push(seat);
      } else {
        this.selectedSeats.splice(index, 1);
      }
    }
  }
};
</script>