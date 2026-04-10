<template>
  <v-card class="pa-4">
    <v-file-input
    label="上傳身體數據 CSV 檔案"
    accept=".csv"
    @change="e => onFileChange(e)"
    prepend-icon="mdi-upload"
    outlined
    class="my-4"
    />
    <v-alert v-if="error" type="error" class="mt-2">{{ error }}</v-alert>
    <v-table v-if="headers.length && rows.length" class="mt-4">
      <thead>
        <tr>
          <th v-for="h in headers" :key="h">{{ h }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, idx) in rows.slice(0, 5)" :key="idx">
          <td v-for="h in headers" :key="h">{{ row[h] }}</td>
        </tr>
      </tbody>
    </v-table>
    <div v-if="rows.length > 5" class="text-caption mt-1">僅顯示前 5 筆資料</div>
  </v-card>
</template>

<script setup>
import { ref } from 'vue'
import Papa from 'papaparse'

const emit = defineEmits(['parsed'])
const headers = ref([])
const rows = ref([])
const error = ref('')

function onFileChange(e) {
  error.value = ''
  let file
  if (e && e.target && e.target.files) {
    file = e.target.files[0]
  } else if (e && e.length) {
    file = e[0]
  } else if (e && e instanceof File) {
    file = e
  }
  if (!file) return
  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      if (results.errors.length) {
        error.value = 'CSV 解析錯誤：' + results.errors[0].message
        return
      }
      headers.value = results.meta.fields
      rows.value = results.data
      emit('parsed', { headers: headers.value, rows: rows.value })
    },
    encoding: 'UTF-8',
  })
}
</script>

<style scoped>
.v-card { max-width: 1000px; margin: 0 auto; }
</style>
