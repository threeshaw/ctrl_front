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

<style scoped>
.excel-table {
  border-collapse: collapse;
  margin-top: 20px;
  width: 100%;
}

.excel-table th,
.excel-table td {
  border: 1px solid #d41111;
  padding: 8px;
  text-align: left;
}

.excel-table {
  background-color: #8ebc45;
  font-weight: bold;
}

button {
  padding: 10px 15px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background-color: #45a049;
}
</style>
