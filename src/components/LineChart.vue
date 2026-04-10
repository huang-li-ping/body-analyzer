<template>
  <div>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import { Chart, registerables } from 'chart.js'
Chart.register(...registerables)

const props = defineProps({
  chartData: Object,
})
const canvas = ref(null)
let chartInstance = null

function renderChart() {
  if (chartInstance) chartInstance.destroy()
  if (!props.chartData?.labels?.length) return
  chartInstance = new Chart(canvas.value, {
    type: 'line',
    data: props.chartData,
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'top' },
        title: { display: false },
      },
      scales: {
        x: { title: { display: true, text: '時間' } },
        y: { title: { display: true, text: '數值' } },
      },
    },
  })
}

watch(() => props.chartData, renderChart, { deep: true })
onMounted(renderChart)
</script>
