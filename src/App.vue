
<script setup>
import { ref } from 'vue'
import CsvUploader from './components/CsvUploader.vue'
import BodyChart from './components/BodyChart.vue'

const headers = ref([])
const rows = ref([])
function onParsed({ headers: h, rows: r }) {
  headers.value = h
  rows.value = r
}
</script>

<template>
  <v-app>
    <v-main>
      <v-container class="py-8">
        <h1 class="mb-6">身體數據分析儀表板</h1>
        <CsvUploader @parsed="onParsed" />
        <BodyChart v-if="headers.length && rows.length" :headers="headers" :rows="rows" />
      </v-container>
    </v-main>
  </v-app>
</template>
