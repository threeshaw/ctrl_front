<template>
  <div>
    <input
      type="file"
      ref="fileInput"
      @change="handleFileChange"
      accept=".xlsx, .xls ,.csv"
      style="display: none"
    />
    <button @click="triggerFileInput">导入Excel</button>
    <div class="content">
      <canvas
        ref="myCanvas"
        height="400"
        width="450"
        @mousedown="onDown"
        @mouseup="onUp"
        @mousemove="onMove"
        @mouseleave="onLeave"
      >
      </canvas>
    </div>
    <div v-if="tableData.length > 0">
      <table class="excel-table">
        <thead>
          <tr>
            <th v-for="(header, index) in tableHeaders" :key="index">
              {{ header || `${index + 1}` }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in tableData" :key="rowIndex">
            <td v-for="(cell, cellIndex) in row" :key="cellIndex">
              {{ cell }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import * as XLSX from 'xlsx'
const canvas = ref(null)

const fileInput = ref(null)
const tableData = ref([])
const tableHeaders = ref([])

// 触发文件选择
const triggerFileInput = () => {
  fileInput.value.click()
}

// 处理文件选择
const handleFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (event) => {
    try {
      const data = new Uint8Array(event.target.result)
      const workbook = XLSX.read(data, { type: 'array' })

      // 获取第一个工作表
      const firstSheetName = workbook.SheetNames[0]
      const worksheet = workbook.Sheets[firstSheetName]

      // 转换为JSON
      const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
      console.log(jsonData[1][2])
      // 处理数据：取前5行5列
      processExcelData(jsonData)
    } catch (error) {
      console.error('Excel解析错误:', error)
      alert('文件解析失败，请检查文件格式')
    }
  }
  reader.readAsArrayBuffer(file)
}
//处理绘图部分的内容

// 处理Excel数据
const processExcelData = (jsonData) => {
  // 获取前5行（如果不足5行则取全部）
  const rows = jsonData.slice(0, 5)

  // 获取表头（第一行）
  tableHeaders.value = rows[0]?.slice(0, 5) || []

  // 处理表格数据（取前5列）
  tableData.value = rows.map((row) =>
    row.slice(0, 5).map((cell) => (cell === null || cell === undefined ? '' : cell)),
  )
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
  background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.container {
  width: 100%;
  max-width: 1000px;
  background-color: rgba(255, 255, 255, 0.93);
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

header {
  background: linear-gradient(90deg, #2c3e50, #4a6491);
  color: white;
  padding: 20px;
  text-align: center;
}

.content {
  display: flex;
  padding: 20px;
  gap: 20px;
}

.chart-container {
  flex: 1;
  position: relative;
  background-color: #f8f9fa;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  padding: 15px;
}

.controls {
  width: 300px;
  background-color: #e9ecef;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.chart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.chart-title {
  font-size: 1.4rem;
  font-weight: 600;
  color: #2c3e50;
}

.zoom-info {
  font-size: 0.9rem;
  color: #6c757d;
}

canvas {
  width: 100%;
  height: 350px;
  background-color: white;
  border-radius: 8px;
  border: 1px solid #dee2e6;
  display: block;
}

.slider-container {
  margin-top: 15px;
  padding: 10px 0;
}

.slider-label {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
  font-size: 0.9rem;
  color: #495057;
}

.range-slider {
  width: 100%;
  height: 8px;
  background: #dee2e6;
  border-radius: 4px;
  outline: none;
}

.range-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4a6491;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.range-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #4a6491;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  border: none;
}

.control-group {
  margin-bottom: 20px;
}

h3 {
  color: #2c3e50;
  margin-bottom: 15px;
  padding-bottom: 8px;
  border-bottom: 2px solid #4a6491;
}

.data-input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 1rem;
  margin-bottom: 15px;
}

.data-input:focus {
  outline: none;
  border-color: #4a6491;
  box-shadow: 0 0 0 3px rgba(74, 100, 145, 0.2);
}

.btn-group {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

button {
  flex: 1;
  padding: 12px 0;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-primary {
  background: linear-gradient(90deg, #4a6491, #2c3e50);
  color: white;
}

.btn-zoom {
  background: linear-gradient(90deg, #3498db, #2980b9);
  color: white;
}

.btn-reset {
  background: linear-gradient(90deg, #e74c3c, #c0392b);
  color: white;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

button:active {
  transform: translateY(0);
}

.instructions {
  background-color: #d1ecf1;
  border-left: 4px solid #0dcaf0;
  padding: 15px;
  border-radius: 6px;
  margin-top: 20px;
}

.instructions h4 {
  color: #0c5460;
  margin-bottom: 10px;
}

.instructions ul {
  padding-left: 20px;
  color: #0c5460;
}

.instructions li {
  margin-bottom: 8px;
}

footer {
  text-align: center;
  padding: 15px;
  background-color: #2c3e50;
  color: white;
  font-size: 0.9rem;
}

@media (max-width: 768px) {
  .content {
    flex-direction: column;
  }

  .controls {
    width: 100%;
  }
}
</style>
