<template>
  <div :class="container">
    <div :class="content">
      <div :class="canvas - container">
        <div :class="canvas - wrapper">
          <canvas
            ref="canvas"
            :width="logicalWidth"
            :height="logicalHeight"
            @mousedown="handleMouseDown"
            @mousemove="handleMouseMove"
            @mouseup="handleMouseUp"
            @mouseleave="handleMouseLeave"
          ></canvas>
        </div>
        <div :class="scale - info">
          缩放比例: {{ scaleRatio.toFixed(2) }} | 逻辑尺寸: {{ logicalWidth }}×{{ logicalHeight }} |
          显示尺寸: {{ displayWidth }}×{{ displayHeight }}
        </div>
      </div>

      <div :class="stats">
        <h2>矩形状态</h2>
        <p>
          逻辑位置: <span class="stat-value">({{ rect.x }}, {{ rect.y }})</span>
        </p>
        <p>
          显示位置:
          <span class="stat-value"
            >({{ displayRectX.toFixed(1) }}, {{ displayRectY.toFixed(1) }})</span
          >
        </p>
        <p>
          当前颜色: <span class="stat-value">{{ rect.colorName }}</span>
        </p>
        <p>
          逻辑尺寸: <span class="stat-value">{{ rect.width }} × {{ rect.height }}</span>
        </p>
        <p>
          显示尺寸:
          <span class="stat-value"
            >{{ displayRectWidth.toFixed(1) }} × {{ displayRectHeight.toFixed(1) }}</span
          >
        </p>
        <p>
          状态: <span class="stat-value">{{ isDragging ? '正在拖动' : '静止' }}</span>
        </p>
      </div>

      <div :class="actions">
        <button @click="resetPosition">重置位置</button>
        <button :class="reset - btn" @click="resetAll">重置全部</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue'
const canvas = ref(null)
const ctx = ref(null)
const displayWidth = ref(0)
const displayHeight = ref(0)
const scaleRatio = ref(1)

// 逻辑尺寸（固定）
const logicalWidth = ref(600)
const logicalHeight = ref(400)

const rect = reactive({
  x: 150,
  y: 120,
  width: 200,
  height: 120,
  color: '#FFD700', // 黄色
  originalColor: '#FFD700',
  originalColorName: '黄色',
  hoverColor: '#FFA500',
  activeColor: '#1E90FF', // 蓝色
  activeColorName: '蓝色',
  isActive: false,
})

// 计算显示尺寸
const displayRectX = computed(() => rect.x * scaleRatio.value)
const displayRectY = computed(() => rect.y * scaleRatio.value)
const displayRectWidth = computed(() => rect.width * scaleRatio.value)
const displayRectHeight = computed(() => rect.height * scaleRatio.value)

// 计算当前颜色名称
const colorName = computed(() => {
  return rect.color === rect.originalColor ? rect.originalColorName : rect.activeColorName
})

// 添加颜色名称到rect对象
rect.colorName = colorName

const isDragging = ref(false)
const isHovering = ref(false)
const beginLining = ref(false)
const stopLining = ref(false)
const dragOffset = reactive({ x: 0, y: 0 })

// 初始化Canvas
const initCanvas = () => {
  if (canvas.value) {
    ctx.value = canvas.value.getContext('2d')
    updateCanvasDimensions()
    draw()
  }
}

// 更新Canvas显示尺寸
const updateCanvasDimensions = () => {
  if (!canvas.value) return

  const container = canvas.value.parentElement
  if (!container) return

  // 获取容器宽度（限制最大宽度）
  const maxWidth = logicalWidth.value
  const containerWidth = Math.min(container.clientWidth, maxWidth)

  // 计算缩放比例
  scaleRatio.value = containerWidth / logicalWidth.value

  // 设置显示尺寸
  displayWidth.value = containerWidth
  displayHeight.value = logicalHeight.value * scaleRatio.value

  // 设置canvas元素的实际显示尺寸
  canvas.value.style.width = `${displayWidth.value}px`
  canvas.value.style.height = `${displayHeight.value}px`
}

// 绘制矩形
const draw = () => {
  if (!ctx.value) return

  // 清除画布
  ctx.value.clearRect(0, 0, logicalWidth.value, logicalHeight.value)

  // 绘制网格背景
  drawGrid()

  // 绘制矩形
  ctx.value.fillStyle = rect.color

  // 拖动时添加阴影和半透明效果
  if (isDragging.value) {
    ctx.value.shadowColor = 'rgba(0, 0, 0, 0.7)'
    ctx.value.shadowBlur = 15
    ctx.value.shadowOffsetX = 5
    ctx.value.shadowOffsetY = 5
    ctx.value.globalAlpha = 0.8
  } else {
    ctx.value.shadowColor = 'transparent'
    ctx.value.shadowBlur = 0
    ctx.value.globalAlpha = 1.0
  }

  // 在逻辑坐标上绘制矩形
  ctx.value.fillRect(rect.x, rect.y, rect.width, rect.height)
  ctx.value.fillStyle = '#00ff1e'
  ctx.value.fillRect(rect.x - 5, rect.y + rect.height / 2 - 5, 10, 10)
  // 悬停时绘制边框
  if (isHovering.value && !isDragging.value) {
    ctx.value.strokeStyle = '#FFFFFF'
    ctx.value.lineWidth = 2
    ctx.value.strokeRect(rect.x - 2, rect.y - 2, rect.width + 4, rect.height + 4)
  }

  // 重置阴影和透明度
  ctx.value.shadowColor = 'transparent'
  ctx.value.shadowBlur = 0
  ctx.value.globalAlpha = 1.0
}

// 绘制网格背景
const drawGrid = () => {
  ctx.value.strokeStyle = 'rgba(100, 100, 150, 0.2)'
  ctx.value.lineWidth = 1

  // 垂直线
  for (let x = 0; x <= logicalWidth.value; x += 20) {
    ctx.value.beginPath()
    ctx.value.moveTo(x, 0)
    ctx.value.lineTo(x, logicalHeight.value)
    ctx.value.stroke()
  }

  // 水平线
  for (let y = 0; y <= logicalHeight.value; y += 20) {
    ctx.value.beginPath()
    ctx.value.moveTo(0, y)
    ctx.value.lineTo(logicalWidth.value, y)
    ctx.value.stroke()
  }
}

// 检查点是否在矩形内（使用逻辑坐标）
const isPointInRect = (logicalX, logicalY) => {
  return (
    logicalX >= rect.x &&
    logicalX <= rect.x + rect.width &&
    logicalY >= rect.y &&
    logicalY <= rect.y + rect.height
  )
}
//依据数组顺序确定判断先后
// 鼠标按下事件
const handleMouseDown = (e) => {
  const canvasBounds = canvas.value.getBoundingClientRect()
  const mouseX = e.clientX - canvasBounds.left
  const mouseY = e.clientY - canvasBounds.top

  // 将显示坐标转换为逻辑坐标
  const logicalX = mouseX / scaleRatio.value
  const logicalY = mouseY / scaleRatio.value

  if (isPointInRect(logicalX, logicalY)) {
    isDragging.value = true

    // 计算拖动偏移量（逻辑坐标）
    dragOffset.x = logicalX - rect.x
    dragOffset.y = logicalY - rect.y

    // 点击矩形切换颜色

    rect.color = rect.activeColor

    draw()
  }
}

// 鼠标移动事件
const handleMouseMove = (e) => {
  const canvasBounds = canvas.value.getBoundingClientRect()
  const mouseX = e.clientX - canvasBounds.left
  const mouseY = e.clientY - canvasBounds.top

  // 将显示坐标转换为逻辑坐标
  const logicalX = mouseX / scaleRatio.value
  const logicalY = mouseY / scaleRatio.value

  // 检查悬停状态
  isHovering.value = isPointInRect(logicalX, logicalY)

  // 处理拖动
  if (isDragging.value) {
    // 计算矩形新位置（逻辑坐标）
    const newX = logicalX - dragOffset.x
    const newY = logicalY - dragOffset.y

    // 限制在画布范围内
    rect.x = Math.max(0, Math.min(logicalWidth.value - rect.width, newX))
    rect.y = Math.max(0, Math.min(logicalHeight.value - rect.height, newY))

    draw()
  } else {
    // 仅当状态变化时重绘
    if (isHovering.value) {
      draw()
    }
  }
}

// 鼠标释放事件
const handleMouseUp = () => {
  isDragging.value = false
  rect.color = rect.originalColor
  draw()
}

// 鼠标离开画布
const handleMouseLeave = () => {
  isHovering.value = false
  if (!isDragging.value) {
    draw()
  }
}

// 重置位置
const resetPosition = () => {
  rect.x = 150
  rect.y = 120
  draw()
}

// 重置全部
const resetAll = () => {
  rect.x = 150
  rect.y = 120
  rect.color = rect.originalColor
  draw()
}

// 窗口大小变化时更新Canvas尺寸
const handleResize = () => {
  updateCanvasDimensions()
  draw()
}

onMounted(() => {
  initCanvas()

  // 添加事件监听器

  // 监听窗口大小变化
  window.addEventListener('resize', handleResize)
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
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
  color: #fff;
}

.container {
  max-width: 900px;
  width: 95%;
  background: rgba(25, 25, 50, 0.85);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  padding: 25px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

header {
  text-align: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid rgba(255, 217, 0, 0.6);
}

h1 {
  font-size: 2.4rem;
  background: linear-gradient(to right, #ffd700, #ffa500);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  margin-bottom: 8px;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.subtitle {
  color: #aaa;
  font-size: 1.1rem;
  margin-top: 5px;
}

.content {
  display: flex;
  flex-wrap: wrap;
  gap: 25px;
}

.canvas-container {
  flex: 1;
  min-width: 300px;
  background: rgba(10, 10, 30, 0.7);
  border-radius: 12px;
  padding: 15px;
  box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.5);
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.canvas-wrapper {
  position: relative;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

canvas {
  background: #121230;
  border-radius: 8px;
  display: block;
  width: 100%;
  height: auto;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  cursor: pointer;
}

.scale-info {
  margin-top: 10px;
  font-size: 0.9rem;
  color: #ccc;
  background: rgba(0, 0, 0, 0.3);
  padding: 5px 10px;
  border-radius: 4px;
}

.controls {
  flex: 0 0 280px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.instructions {
  background: rgba(30, 30, 60, 0.7);
  padding: 18px;
  border-radius: 12px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.instructions h2 {
  color: #ffd700;
  margin-bottom: 12px;
  font-size: 1.4rem;
}

.instructions ul {
  padding-left: 18px;
}

.instructions li {
  margin-bottom: 8px;
  line-height: 1.5;
  font-size: 0.95rem;
}

.stats {
  background: rgba(30, 30, 60, 0.7);
  padding: 18px;
  border-radius: 12px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.stats h2 {
  color: #ffd700;
  margin-bottom: 12px;
  font-size: 1.4rem;
}

.stats p {
  margin: 8px 0;
  font-size: 1rem;
}

.stat-value {
  color: #ffd700;
  font-weight: bold;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 10px;
}

button {
  flex: 1;
  min-width: 110px;
  padding: 10px 16px;
  border: none;
  border-radius: 8px;
  background: linear-gradient(to right, #ffa500, #ff8c00);
  color: #1a1a2e;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.95rem;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

button:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.4);
  background: linear-gradient(to right, #ffd700, #ffa500);
}

button:active {
  transform: translateY(1px);
}

.reset-btn {
  background: linear-gradient(to right, #3498db, #2980b9);
}

.footer {
  text-align: center;
  margin-top: 25px;
  padding-top: 15px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  color: #aaa;
  font-size: 0.85rem;
}

.responsive-label {
  position: absolute;
  top: 15px;
  right: 15px;
  background: #27ae60;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-weight: bold;
  animation: pulse 1.5s infinite;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@media (max-width: 768px) {
  .content {
    flex-direction: column;
  }

  .controls {
    flex: 1;
    width: 100%;
  }

  h1 {
    font-size: 2rem;
  }

  .container {
    padding: 20px 15px;
  }
}

@media (max-width: 480px) {
  h1 {
    font-size: 1.8rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  button {
    min-width: 100%;
  }
}
</style>
