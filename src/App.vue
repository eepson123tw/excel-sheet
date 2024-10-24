<script setup>
import ImportExcel from '@/components/ImportExcel.vue'
import ExcelShowCase from '@/components/ExcelShowCase.vue'
import { ref } from 'vue'

const file = ref([])

const uploadFile = newFile => {
  file.value.push(newFile)
}
</script>

<template>
  <div class="wrapper">
    <ImportExcel @uploadFile="uploadFile"></ImportExcel>
    <ExcelShowCase :file="file"></ExcelShowCase>
  </div>
</template>
<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
}
</style>
