<script setup>
import { watch, ref, onBeforeUnmount } from 'vue'
import * as XLSX from 'xlsx'
import Spreadsheet from 'x-data-spreadsheet'
import 'x-data-spreadsheet/dist/xspreadsheet.css' // Import CSS

// Define props
const props = defineProps({
  file: {
    type: Array,
    required: true,
  },
  options: {
    type: Object,
    default: () => ({}),
  },
})

// State
const parsedData = ref({})
const sheetEl = ref(null)
let spreadsheetInstance = null

// Transform XLSX JSON to x-data-spreadsheet format
function transformToSpreadsheetData(jsonData) {
  return {
    rows: jsonData.map(row => ({
      cells: row.map(cell => ({ text: cell != null ? String(cell) : '' })),
    })),
  }
}

// Convert spreadsheet data back to JSON
function dataToJson(data) {
  return data.rows.map(row => row.cells.map(cell => cell.text))
}

// Initialize or update the spreadsheet
function initializeSpreadsheet(data) {
  if (spreadsheetInstance) {
    spreadsheetInstance.destroy()
  }

  spreadsheetInstance = new Spreadsheet('#x-spreadsheet-demo', {
    view: {
      height: () =>
        (sheetEl.value && sheetEl.value.clientHeight) ||
        document.documentElement.clientHeight,
      width: () =>
        (sheetEl.value && sheetEl.value.clientWidth) ||
        document.documentElement.clientWidth,
    },
    ...props.options,
  })
    .loadData(data)
    .change(newData => {
      // Update parsedData on changes
      const firstFileName = Object.keys(parsedData.value)[0]
      if (firstFileName) {
        parsedData.value[firstFileName] = dataToJson(newData)
      }
    })
}

// Watch for file changes
watch(
  () => props.file,
  newFiles => {
    console.log('New files:', newFiles)
    if (!newFiles.length) return

    // Clear existing parsed data
    parsedData.value = {}

    newFiles.forEach(file => {
      const reader = new FileReader()

      reader.onload = e => {
        try {
          const arrayBuffer = e.target.result
          const data = new Uint8Array(arrayBuffer)
          const workbook = XLSX.read(data, { type: 'array' })

          const firstSheetName = workbook.SheetNames[0]
          const worksheet = workbook.Sheets[firstSheetName]

          const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
          console.log(`Data for file ${file.name}:`, jsonData)

          parsedData.value[file.name] = jsonData

          // Initialize or update the spreadsheet with the first file's data
          const transformedData = transformToSpreadsheetData(jsonData)
          initializeSpreadsheet(transformedData)
        } catch (error) {
          console.error(`Error processing file ${file.name}:`, error)
        }
      }

      reader.onerror = error => {
        console.error(`Error reading file ${file.name}:`, error)
      }

      reader.readAsArrayBuffer(file)
    })
  },
  { immediate: true, deep: true },
)

// Cleanup on unmount
onBeforeUnmount(() => {
  if (spreadsheetInstance) {
    spreadsheetInstance.destroy()
    spreadsheetInstance = null
  }
})
</script>

<template>
  <div class="showCase">
    <h2>Showcase</h2>
    <ul>
      <li v-for="(f, index) in props.file" :key="index">{{ f.name }}</li>
    </ul>
    <div
      ref="sheetEl"
      id="x-spreadsheet-demo"
      style="width: 800px; height: 500px"
    ></div>
    <!-- Optional: Display parsed data -->
  </div>
  <div class="showData">
    <div v-for="(data, fileName) in parsedData" :key="fileName">
      <h3>{{ fileName }}</h3>
      <pre>{{ data }}</pre>
    </div>
  </div>
</template>

<style scoped>
.showCase {
  width: 1100px;
  /* Add your styles here */
}
.showData {
  width: 500px;
  overflow: hidden;
  overflow-x: scroll;
}
</style>
