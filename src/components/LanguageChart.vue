<script setup>
import { Chart as ChartJS, ArcElement, Tooltip, Legend } from 'chart.js'
import { Pie } from 'vue-chartjs'
import { computed } from 'vue'

// Registramos os componentes do gráfico para funcionarem
ChartJS.register(ArcElement, Tooltip, Legend)

// Recebemos os dados prontos do componente pai
const props = defineProps({
  languages: {
    type: Object,
    required: true
  }
})

// Configuração dos dados para o Chart.js entender
const chartData = computed(() => {
  return {
    labels: Object.keys(props.languages), // Ex: ['JavaScript', 'HTML', 'Vue']
    datasets: [
      {
        backgroundColor: ['#41B883', '#E46651', '#00D8FF', '#DD1B16', '#F7DF1E'],
        data: Object.values(props.languages)
      }
    ]
  }
})

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false
}
</script>

<template>
  <div class="chart-container">
    <h3>Linguagens Mais Usadas</h3>
    <div v-if="Object.keys(languages).length === 0">Sem dados suficientes</div>
    <Pie v-else :data="chartData" :options="chartOptions" />
  </div>
</template>

<style scoped>
.chart-container {
  background: rgb(179, 198, 210);
  box-shadow: 0 4px 12px rgba(227, 224, 224, 0.1);
  text-align: center;
  min-height: 400px; 
  padding: 20px;
  display: flex;
  border-radius: 12px;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
h3 { margin-bottom: 15px; color: #fdfeff; font-weight: 600; }

div canvas {
  max-width: 100%;
  max-height: 300px;
}
</style>