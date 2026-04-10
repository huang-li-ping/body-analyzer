<template>
  <v-card class="pa-4 mt-4">
    <v-row>
      <v-col cols="12" md="4">
        <v-select
          v-model="selectedFields"
          :items="fields"
          label="選擇要顯示的指標 (可複選)"
          multiple
          chips
        />
      </v-col>
      <v-col cols="12" md="4">
        <v-select
          v-model="timeFilter"
          :items="timeFilterOptions"
          label="篩選上午/下午/全部"
        />
      </v-col>
    </v-row>
    <line-chart v-if="selectedFields.length && chartData.labels.length" :chart-data="chartData" />
    <div v-else class="mt-4" style="color:#888;">請先選擇要顯示的指標</div>
  </v-card>
</template>

<script setup>
import { ref, watch, computed } from 'vue'
import LineChart from './LineChart.vue'

const props = defineProps({
  headers: Array,
  rows: Array,
})



const fields = computed(() => props.headers?.filter(h => h !== '時間') || [])
const selectedFields = ref([])

const timeFilterOptions = [
  { title: '全部', value: 'all' },
  { title: '上午', value: 'am' },
  { title: '下午', value: 'pm' },
]
const timeFilter = ref('all')

const chartData = computed(() => {
  if (!props.rows?.length || !selectedFields.value.length) return { labels: [], datasets: [] }
  // 依照時間排序（舊到新）
  const parse = s => {
    if (!s || typeof s !== 'string') return 0
    const [time, date] = s.split(' ')
    if (!time || !date) return 0
    const [h = '00', m = '00'] = (time || '').split(':')
    const [d = '01', M = '01', Y = '1970'] = (date || '').split('/')
    return new Date(`${Y}-${M.padStart(2, '0')}-${d.padStart(2, '0')}T${h.padStart(2, '0')}:${m.padStart(2, '0')}`)
  }
  let filteredRows = props.rows
  if (timeFilter.value === 'am') {
    filteredRows = filteredRows.filter(r => {
      const t = (r['時間']||'').split(' ')[0]
      const h = parseInt((t||'').split(':')[0], 10)
      return h >= 0 && h < 12
    })
  } else if (timeFilter.value === 'pm') {
    filteredRows = filteredRows.filter(r => {
      const t = (r['時間']||'').split(' ')[0]
      const h = parseInt((t||'').split(':')[0], 10)
      return h >= 12 && h < 24
    })
  }
  const sortedRows = [...filteredRows].sort((a, b) => parse(a['時間']) - parse(b['時間']))
  const labels = sortedRows.map(r => r['時間'])
  const datasets = selectedFields.value.map((field, idx) => ({
    label: field,
    data: sortedRows.map(r => parseFloat((r[field]||'').replace(/[^\d.\-]/g, '')) || 0),
    borderColor: colors[idx % colors.length],
    backgroundColor: colors[idx % colors.length] + '33',
    tension: 0.3,
    pointRadius: 2,
  }))
  return { labels, datasets }
})

const colors = [
  '#1976D2', '#E53935', '#43A047', '#FB8C00', '#8E24AA', '#00ACC1', '#FDD835', '#6D4C41', '#1E88E5', '#D81B60'
]
</script>

<style scoped>
.v-card { max-width: 1000px; margin: 0 auto; }
</style>
