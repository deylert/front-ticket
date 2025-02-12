<template>
      <container> <!-- Ajusta la altura aquí -->
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
 
 </style>