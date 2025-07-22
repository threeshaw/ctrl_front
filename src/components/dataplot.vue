<template>
  <div id="app">
    <div class="container">
      <div class="content">
        <div class="chart-container">
          <div class="chart-header">
            <div class="chart-title">数据趋势分析</div>
            <div class="zoom-info">缩放比例: {{ zoomLevel.toFixed(2) }}x</div>
          </div>
          <canvas ref="chartCanvas"></canvas>
          <div class="slider-container">
            <div class="slider-label">
              <span>显示范围</span>
              <span>{{ rangeLabel }}</span>
            </div>
            <input
              ref="datarange"
              type="range"
              :min="0"
              :max="20"
              v-model="sliderValue"
              class="range-slider"
            />
          </div>
        </div>

        <div class="controls">
          <div class="control-group">
            <h3>数据输入</h3>
            <input
              type="text"
              v-model="inputData"
              class="data-input"
              placeholder="输入数据点，用逗号分隔 (例如: 12, 18, 22, 15, 30)"
            />
            <button @click="plotChart" class="btn-primary">导入数据</button>
          </div>

          <div class="control-group">
            <h3>缩放控制</h3>
            <div class="btn-group">
              <button @click="zoomIn" class="btn-zoom">放大 (+)</button>
              <button @click="zoomOut" class="btn-zoom">缩小 (-)</button>
            </div>
            <button @click="resetView" class="btn-reset">重置视图</button>
          </div>

          <div class="instructions">
            <h4>使用说明：</h4>
            <ul>
              <li>在输入框中输入数字，用逗号分隔</li>
              <li>点击"绘制图表"按钮生成折线图</li>
              <li>使用"放大"和"缩小"按钮调整图表显示范围</li>
              <li>使用底部滑块浏览不同数据段</li>
              <li>点击"重置视图"恢复原始视图</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { createApp, ref, reactive, onMounted, watch } from 'vue'

// 响应式数据
const chartCanvas = ref(null)
const inputData = ref('')
const zoomLevel = ref(1.0)
const startIndex = ref(0)
const sliderValue = ref(0)
const rangeLabel = ref('0 - 20')
const datarange = ref('')

// 图表数据
const data = reactive({
  values: [12, 19, 15, 22, 17, 25, 19, 30, 27, 35, 31, 40, 38, 45, 42, 48, 45, 50, 47, 42],
  visible: [],
})

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
  console.log(endIndex)
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
    ctx.fillText(dataIndex.toString(), x, padding.top + chartHeight + 20)
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
  ctx.translate(20, chartCanvas.value.height / 2)
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
    20,
  )

  // 更新范围标签
  rangeLabel.value = `${startIndex.value} - ${endIndex}`

  // 更新滑块最大值
  updateRangeSlider()
}
function updateRangeSlider() {
  datarange.min = 0
  datarange.max = Math.max(0, data.values.length - data.visible.length)
  console.log(datarange.max)
  datarange.value = startIndex
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
body {
  background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
  min-height: 100vh;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.container {
  width: 1000px;
  background-color: rgba(255, 255, 255, 0.93);
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.content {
  display: flex;
  padding: 20px;
  gap: 20px;
}

.chart-container {
  width: 800;
  position: relative;
  background-color: #f8f9fa;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  padding: 15px;
}

.controls {
  width: 800px;
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
  height: 550px;
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
</style>
