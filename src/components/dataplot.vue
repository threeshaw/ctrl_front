<template>
  <div class="container">
    <canvas ref="chartCanvas"></canvas>
    <div class="slider-container">
      <div class="slider-label">
        <span>显示范围</span>
        <span>{{ rangeLabel }}</span>
      </div>
      <div class="controls">
        <div class="control">
          <input
            type="file"
            ref="fileInput"
            @change="handleFileChange"
            accept=".xlsx, .xls ,.csv"
            style="display: none"
          />
          <button @click="triggerFileInput">导入Excel</button>
          <button @click="plotChart" class="btn-primary">绘制图表</button>
        </div>

        <div class="control-group">
          <div class="btn-group">
            <button @click="zoomIn" class="btn-zoom">放大 (+)</button>
            <button @click="zoomOut" class="btn-zoom">缩小 (-)</button>
          </div>
          <button @click="resetView" class="btn-reset">重置视图</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, watch } from 'vue'
import * as XLSX from 'xlsx' // 响应式数据
const chartCanvas = ref(null)
const inputData = ref('')
const zoomLevel = ref(1.0)
const startIndex = ref(0)
const sliderValue = ref(0)
const sliderMax = ref(0)
const rangeLabel = ref('0 - 100')
const fileInput = ref(null)
// 触发文件选择
const triggerFileInput = () => {
  fileInput.value.click()
}
const data = reactive({
  values: [12, 19, 15, 22, 17, 25, 19, 30, 27, 35, 31, 40, 38, 45, 42, 48, 45, 50, 47, 42],
  visible: [],
})
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
      data.values = jsonData.slice(0, 100)
      // 处理数据：取前5行5列
      processExcelData(jsonData)
    } catch (error) {
      console.error('Excel解析错误:', error)
      alert('文件解析失败，请检查文件格式')
    }
  }
  reader.readAsArrayBuffer(file)
}
// 图表数据

// 图表配置
const padding = reactive({ top: 40, right: 30, bottom: 50, left: 60 })
let chartWidth = 0
let chartHeight = 0
let ctx = null

// 初始化图表
const initChart = () => {
  if (!chartCanvas.value) return

  const canvas = chartCanvas.value
  canvas.width = canvas.offsetWidth
  canvas.height = canvas.offsetHeight
  ctx = canvas.getContext('2d')

  chartWidth = canvas.width - padding.left - padding.right
  chartHeight = canvas.height - padding.top - padding.bottom

  drawChart()
}

// 绘制图表
const drawChart = () => {
  if (!ctx) return

  // 清除Canvas
  ctx.clearRect(0, 0, chartCanvas.value.width, chartCanvas.value.height)

  // 绘制背景
  ctx.fillStyle = '#ffffff'
  ctx.fillRect(0, 0, chartCanvas.value.width, chartCanvas.value.height)

  // 计算可见数据点
  const visiblePoints = Math.min(
    data.values.length,
    Math.ceil(data.values.length / zoomLevel.value),
  )
  const endIndex = Math.min(startIndex.value + visiblePoints, data.values.length)
  data.visible = data.values.slice(startIndex.value, endIndex)

  // 计算数据范围
  const maxValue = Math.max(...data.visible) * 1.1
  const minValue = Math.min(0, Math.min(...data.visible) * 0.9)

  // 计算比例因子
  const xScale = chartWidth / (data.visible.length - 1)
  const yScale = chartHeight / (maxValue - minValue)

  // 绘制网格
  ctx.strokeStyle = '#e0e0e0'
  ctx.lineWidth = 1
  ctx.beginPath()

  // 水平网格线
  const yGridLines = 8
  for (let i = 0; i <= yGridLines; i++) {
    const y = padding.top + chartHeight - (i / yGridLines) * chartHeight
    ctx.moveTo(padding.left, y)
    ctx.lineTo(padding.left + chartWidth, y)

    // 绘制刻度标签
    ctx.fillStyle = '#6c757d'
    ctx.font = '12px Arial'
    ctx.textAlign = 'right'
    ctx.fillText(
      Math.round(minValue + (i / yGridLines) * (maxValue - minValue)).toString(),
      padding.left - 10,
      y + 4,
    )
  }

  // 垂直网格线
  const xGridLines = Math.min(12, data.visible.length)
  for (let i = 0; i < xGridLines; i++) {
    const x = padding.left + (i / (xGridLines - 1)) * chartWidth
    ctx.moveTo(x, padding.top)
    ctx.lineTo(x, padding.top + chartHeight)

    // 绘制刻度标签
    ctx.fillStyle = '#6c757d'
    ctx.font = '12px Arial'
    ctx.textAlign = 'center'
    const dataIndex =
      startIndex.value + Math.round((i * (data.visible.length - 1)) / (xGridLines - 1))
    ctx.fillText(dataIndex.toString(), x, padding.top + chartHeight + 100)
  }

  ctx.stroke()

  // 绘制坐标轴
  ctx.strokeStyle = '#2c3e50'
  ctx.lineWidth = 2
  ctx.beginPath()

  // Y轴
  ctx.moveTo(padding.left, padding.top)
  ctx.lineTo(padding.left, padding.top + chartHeight)

  // X轴
  ctx.moveTo(padding.left, padding.top + chartHeight)
  ctx.lineTo(padding.left + chartWidth, padding.top + chartHeight)

  ctx.stroke()

  // 绘制轴标签
  ctx.fillStyle = '#2c3e50'
  ctx.font = 'bold 14px Arial'
  ctx.textAlign = 'center'
  ctx.fillText('数据点索引', chartCanvas.value.width / 2, chartCanvas.value.height - 15)

  ctx.save()
  ctx.translate(100, chartCanvas.value.height / 2)
  ctx.rotate(-Math.PI / 2)
  ctx.textAlign = 'center'
  ctx.fillText('数值', 0, 0)
  ctx.restore()

  // 绘制数据点折线
  ctx.beginPath()
  ctx.moveTo(padding.left, padding.top + chartHeight - (data.visible[0] - minValue) * yScale)
  ctx.strokeStyle = '#e74c3c'
  ctx.lineWidth = 3

  for (let i = 1; i < data.visible.length; i++) {
    const x = padding.left + i * xScale
    const y = padding.top + chartHeight - (data.visible[i] - minValue) * yScale
    ctx.lineTo(x, y)
  }

  ctx.stroke()

  // 绘制数据点
  ctx.fillStyle = '#e74c3c'
  for (let i = 0; i < data.visible.length; i++) {
    const x = padding.left + i * xScale
    const y = padding.top + chartHeight - (data.visible[i] - minValue) * yScale

    ctx.beginPath()
    ctx.arc(x, y, 5, 0, Math.PI * 2)
    ctx.fill()

    // 在点上方显示数值
    if (data.visible.length <= 30) {
      ctx.fillStyle = '#2c3e50'
      ctx.font = '12px Arial'
      ctx.textAlign = 'center'
      ctx.fillText(data.visible[i], x, y - 15)
      ctx.fillStyle = '#e74c3c'
    }
  }

  // 绘制标题
  ctx.fillStyle = '#2c3e50'
  ctx.font = 'bold 16px Arial'
  ctx.textAlign = 'center'
  ctx.fillText(
    `数据趋势分析 (${startIndex.value}-${endIndex}/${data.values.length} 个数据点)`,
    chartCanvas.value.width / 2,
    100,
  )

  // 更新范围标签
  rangeLabel.value = `${startIndex.value} - ${endIndex}`

  // 更新滑块最大值
  sliderMax.value = Math.max(0, data.values.length - data.visible.length)
}

// 绘制图表按钮
const plotChart = () => {
  if (inputData.value) {
    // 解析输入数据
    const newData = inputData.value
      .split(',')
      .map((num) => parseFloat(num.trim()))
      .filter((num) => !isNaN(num))
    if (newData.length > 0) {
      data.values = newData
      resetView()
    } else {
      alert('请输入有效的数字数据！')
    }
  } else {
    // 使用默认数据
    data.values = [12, 19, 15, 22, 17, 25, 19, 30, 27, 35, 31, 40, 38, 45, 42, 48, 45, 50, 47, 42]
    resetView()
  }
}

// 放大图表
const zoomIn = () => {
  if (zoomLevel.value < 5.0) {
    zoomLevel.value += 0.2
    // 当放大时，保持当前视图中心
    const currentCenter = startIndex.value + data.visible.length / 2
    startIndex.value = Math.max(
      0,
      Math.min(
        currentCenter - Math.ceil(data.visible.length / 2),
        data.values.length - data.visible.length,
      ),
    )
    drawChart()
  }
}

// 缩小图表
const zoomOut = () => {
  if (zoomLevel.value > 0.2) {
    zoomLevel.value -= 0.2
    // 当缩小时，保持当前视图中心
    const currentCenter = startIndex.value + data.visible.length / 2
    startIndex.value = Math.max(
      0,
      Math.min(
        currentCenter - Math.ceil(data.visible.length / 2),
        data.values.length - data.visible.length,
      ),
    )
    drawChart()
  }
}

// 重置视图
const resetView = () => {
  zoomLevel.value = 1.0
  startIndex.value = 0
  sliderValue.value = 0
  drawChart()
}

// 监听滑块变化
watch(sliderValue, (newVal) => {
  startIndex.value = newVal
  drawChart()
})

// 组件挂载时初始化图表
onMounted(() => {
  initChart()
  // 监听窗口大小变化
  window.addEventListener('resize', initChart)
})
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
</style>
