<script setup>
const emit = defineEmits(['uploadFile'])

function handleDragover(e) {
  e.stopPropagation()
  e.preventDefault()
  e.dataTransfer.dropEffect = 'copy'
}

function handleDrop(e) {
  e.stopPropagation()
  e.preventDefault()
  const files = e.dataTransfer.files
  if (files.length) {
    emit('uploadFile', files[0])
  }
}

function importExcel(e) {
  const file = e.target.files[0]
  emit('uploadFile', file)
}
</script>

<template>
  <div class="greetings">
    <p>Let's import an Excel file!</p>
    <input type="file" @change="importExcel" />
    <div
      class="dropZone"
      @dragover="handleDragover"
      @drop="handleDrop"
      @dragenter="handleDragover"
    >
      <p>Or drag and drop it here to see sheet data</p>
    </div>
  </div>
</template>

<style scoped>
.dropZone {
  border: 2px dashed #bbb;
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
  padding: 25px;
  text-align: center;
  font:
    20pt bold,
    'Vollkorn';
  color: #bbb;
}
</style>
