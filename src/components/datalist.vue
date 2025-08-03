<template>
  <div class="modal-overlay" v-if="showModal">
    <div class="modal-content">
      <div class="modal-header">
        <h2>{{ modalTitle }}</h2>
        <button class="close-button" @click="closeModal">&times;</button>
      </div>

      <div class="form-group">
        <label>数据集名称</label>
        <input type="text" v-model="newFile.name" placeholder="输入数据集名称" />
      </div>

      <div class="form-group">
        <label>数据集描述</label>
        <textarea v-model="newFile.description" placeholder="输入数据集描述"></textarea>
      </div>

      <div class="form-group">
        <label>上传CSV文件</label>
        <div class="file-upload" @click="triggerFileInput" @dragover.prevent @drop="handleDrop">
          <i>📁</i>
          <p>点击或拖拽CSV文件到此处</p>
          <p v-if="newFile.file">已选择: {{ newFile.file.name }}</p>
          <input
            type="file"
            ref="fileInput"
            style="display: none"
            accept=".csv"
            @change="handleFileUpload"
          />
        </div>
      </div>

      <div class="modal-footer">
        <button class="secondary" @click="closeModal">取消</button>
        <button @click="addNewFile">确认添加</button>
      </div>
    </div>
  </div>

  <!-- 主应用 -->
  <div class="containerleft">
    <ul class="mainlist">
      <div @click="changeState1">过程数据</div>
      <div v-for="item in list1" v-if="show1">
        {{ item }}
      </div>
    </ul>
    <ul class="mainlist">
      <div @click="changeState2">过程变量</div>
      <div v-for="item in list2" v-if="show2">
        {{ item }}
      </div>
    </ul>
    <ul class="mainlist">
      <div @click="changeState2">过程曲线</div>
      <div v-for="item in list3" v-if="show3">
        {{ item }}
      </div>
    </ul>
  </div>
  <div class="container">
    <div class="main-content">
      <div class="file-list">
        <div class="file-list-header">
          <h2>数据集列表</h2>
          <button class="secondary" @click="openNewFileModal">+ 新建文件</button>
        </div>

        <div
          v-for="(file, index) in files"
          :key="index"
          class="file-item"
          :class="{ active: activeFileIndex === index }"
          @dblclick="loadFile(index)"
        >
          <h3>
            {{ file.name }}
            <span class="edit-icon" @click.stop="editFile(index)">✎</span>
          </h3>
          <p>{{ file.description }}</p>
          <div class="file-meta">
            <span>数据点: {{ file.data.length }}</span>
            <span>更新于: {{ file.updated }}</span>
          </div>
        </div>

        <p v-if="files.length === 0" style="color: #90a4ae; text-align: center; margin-top: 20px">
          暂无数据集，请点击"新建文件"按钮添加
        </p>
      </div>

      <div class="visualization-area">
        <div class="canvas-container">
          <canvas ref="canvas" width="800" height="600"></canvas>
          <div v-if="!currentData.length" class="no-data">
            <p>请双击左侧文件加载数据</p>
            <p>或使用"生成随机数据"按钮</p>
          </div>
        </div>

        <div class="controls">
          <div class="info-panel">
            <h3>数据信息</h3>
            <div v-if="currentData.length">
              <p>当前数据集: {{ activeFileName }}</p>
              <div class="stats">
                <div class="stat-item">
                  <h4>数据点数量</h4>
                  <div class="stat-value">{{ currentData.length }}</div>
                </div>
                <div class="stat-item">
                  <h4>X轴范围</h4>
                  <div class="stat-value">
                    {{ xRange[0].toFixed(2) }} - {{ xRange[1].toFixed(2) }}
                  </div>
                </div>
                <div class="stat-item">
                  <h4>Y轴范围</h4>
                  <div class="stat-value">
                    {{ yRange[0].toFixed(2) }} - {{ yRange[1].toFixed(2) }}
                  </div>
                </div>
              </div>
            </div>
            <p v-else>未加载任何数据</p>
          </div>

          <div class="button-group">
            <button @click="generateRandomData">下一步</button>
            <button @click="clearCanvas">清除画布</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, onMounted, computed } from 'vue'
import { RouterLink, RouterView } from 'vue-router'
const canvas = ref(null)
const ctx = ref(null)
const fileInput = ref(null)

// 文件数据
const files = ref([
  {
    name: '用户集',
    description: '用户分析',
    data: generateSampleData(150, 0, 100, 0, 100),
    updated: getCurrentDate(),
  },
  {
    name: '数据',
    description: '关系',
    data: generateSampleData(200, 20, 500, 5, 200),
    updated: getCurrentDate(),
  },
  {
    name: '数据集',
    description: '读数',
    data: generateSampleData(120, 10, 50, 20, 90),
    updated: getCurrentDate(),
  },
])

const activeFileIndex = ref(-1)
const currentData = ref([])
const showModal = ref(false)
const modalTitle = ref('新建数据集')
const editingIndex = ref(-1)

// 新文件数据模型
const newFile = ref({
  name: '',
  description: '',
  file: null,
  data: [],
})

// 计算数据范围
const xRange = computed(() => {
  if (!currentData.value.length) return [0, 0]
  const xs = currentData.value.map((p) => p[0])
  return [Math.min(...xs), Math.max(...xs)]
})

const yRange = computed(() => {
  if (!currentData.value.length) return [0, 0]
  const ys = currentData.value.map((p) => p[1])
  return [Math.min(...ys), Math.max(...ys)]
})

const activeFileName = computed(() => {
  return activeFileIndex.value >= 0 ? files.value[activeFileIndex.value].name : ''
})

// 打开新建文件模态框
const openNewFileModal = () => {
  resetNewFile()
  modalTitle.value = '新建数据集'
  editingIndex.value = -1
  showModal.value = true
}

// 编辑文件
const editFile = (index) => {
  const file = files.value[index]
  newFile.value = {
    name: file.name,
    description: file.description,
    file: null,
    data: [...file.data],
  }
  modalTitle.value = '编辑数据集'
  editingIndex.value = index
  showModal.value = true
}

// 关闭模态框
const closeModal = () => {
  showModal.value = false
}

// 重置新文件数据
const resetNewFile = () => {
  newFile.value = {
    name: '',
    description: '',
    file: null,
    data: [],
  }
}

// 触发文件选择
const triggerFileInput = () => {
  fileInput.value.click()
}

// 处理文件上传
const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (file && file.name.endsWith('.csv')) {
    newFile.value.file = file
    parseCSV(file)
  } else {
    alert('请上传有效的CSV文件')
  }
}

// 处理文件拖放
const handleDrop = (event) => {
  event.preventDefault()
  const file = event.dataTransfer.files[0]
  if (file && file.name.endsWith('.csv')) {
    newFile.value.file = file
    parseCSV(file)
  } else {
    alert('请拖放有效的CSV文件')
  }
}

// 解析CSV文件
const parseCSV = (file) => {
  const reader = new FileReader()
  reader.onload = (e) => {
    const content = e.target.result
    const lines = content.split('\n').filter((line) => line.trim() !== '')
    const data = []

    // 跳过标题行，从第二行开始解析
    for (let i = 1; i < lines.length; i++) {
      const parts = lines[i].split(',').map((part) => part.trim())
      if (parts.length >= 2) {
        const x = parseFloat(parts[0])
        const y = parseFloat(parts[1])
        if (!isNaN(x) && !isNaN(y)) {
          data.push([x, y])
        }
      }
    }

    newFile.value.data = data
    if (!newFile.value.name) {
      newFile.value.name = file.name.replace('.csv', '')
    }
  }
  reader.readAsText(file)
}

// 添加新文件
const addNewFile = () => {
  if (!newFile.value.name) {
    alert('请输入数据集名称')
    return
  }

  if (newFile.value.data.length === 0) {
    alert('请上传有效的CSV文件')
    return
  }

  const newFileObj = {
    name: newFile.value.name,
    description: newFile.value.description || '无描述信息',
    data: newFile.value.data,
    updated: getCurrentDate(),
  }

  if (editingIndex.value >= 0) {
    // 编辑现有文件
    files.value[editingIndex.value] = newFileObj
  } else {
    // 添加新文件
    files.value.push(newFileObj)
  }

  showModal.value = false
  resetNewFile()
}

// 加载文件数据
const loadFile = (index) => {
  activeFileIndex.value = index
  currentData.value = [...files.value[index].data]
  drawScatterPlot()
}

// 绘制散点图
const drawScatterPlot = () => {
  if (!ctx.value || !currentData.value.length) return

  const canvasEl = canvas.value
  ctx.value.clearRect(0, 0, canvasEl.width, canvasEl.height)

  // 设置画布尺寸
  const width = canvasEl.width
  const height = canvasEl.height
  const padding = 50

  // 计算比例因子
  const xMin = xRange.value[0]
  const xMax = xRange.value[1]
  const yMin = yRange.value[0]
  const yMax = yRange.value[1]

  const scaleX = (width - 2 * padding) / (xMax - xMin)
  const scaleY = (height - 2 * padding) / (yMax - yMin)

  // 绘制坐标轴
  ctx.value.strokeStyle = '#4fc3f7'
  ctx.value.lineWidth = 2
  ctx.value.beginPath()
  ctx.value.moveTo(padding, padding)
  ctx.value.lineTo(padding, height - padding)
  ctx.value.lineTo(width - padding, height - padding)
  ctx.value.stroke()

  // 绘制网格
  ctx.value.strokeStyle = 'rgba(79, 195, 247, 0.2)'
  ctx.value.lineWidth = 1

  // X轴网格
  for (let i = 1; i < 10; i++) {
    const x = padding + (i * (width - 2 * padding)) / 10
    ctx.value.beginPath()
    ctx.value.moveTo(x, padding)
    ctx.value.lineTo(x, height - padding)
    ctx.value.stroke()
  }

  // Y轴网格
  for (let i = 1; i < 10; i++) {
    const y = padding + (i * (height - 2 * padding)) / 10
    ctx.value.beginPath()
    ctx.value.moveTo(padding, y)
    ctx.value.lineTo(width - padding, y)
    ctx.value.stroke()
  }

  // 绘制点
  ctx.value.fillStyle = '#FF5722'
  for (const point of currentData.value) {
    const x = padding + (point[0] - xMin) * scaleX
    const y = height - padding - (point[1] - yMin) * scaleY

    ctx.value.beginPath()
    ctx.value.arc(x, y, 5, 0, Math.PI * 2)
    ctx.value.fill()
  }

  // 绘制标题
  ctx.value.fillStyle = '#ffffff'
  ctx.value.font = '16px Arial'
  ctx.value.textAlign = 'center'
  ctx.value.fillText(activeFileName.value, width / 2, 30)

  // 绘制坐标轴标签
  ctx.value.font = '12px Arial'
  ctx.value.fillStyle = '#90a4ae'

  // X轴标签
  ctx.value.textAlign = 'center'
  ctx.value.fillText('X 轴', width / 2, height - 10)

  // Y轴标签
  ctx.value.save()
  ctx.value.translate(15, height / 2)
  ctx.value.rotate(-Math.PI / 2)
  ctx.value.textAlign = 'center'
  ctx.value.fillText('Y 轴', 0, 0)
  ctx.value.restore()
}

// 生成随机数据
const generateRandomData = () => {
  activeFileIndex.value = -1
  currentData.value = generateSampleData(Math.floor(Math.random() * 100) + 50, 0, 100, 0, 100)
  drawScatterPlot()
}

// 清除画布
const clearCanvas = () => {
  activeFileIndex.value = -1
  currentData.value = []
  if (ctx.value) {
    ctx.value.clearRect(0, 0, canvas.value.width, canvas.value.height)
  }
}

// 生成示例数据
function generateSampleData(count, minX, maxX, minY, maxY) {
  const data = []
  for (let i = 0; i < count; i++) {
    const x = minX + Math.random() * (maxX - minX)
    const y = minY + Math.random() * (maxY - minY)
    data.push([x, y])
  }
  return data
}

// 获取当前日期
function getCurrentDate() {
  const now = new Date()
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')}`
}

onMounted(() => {
  ctx.value = canvas.value.getContext('2d')
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
  background-color: white;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.containerleft {
  width: 200px;
  height: auto;
  background-color: rgb(20, 9, 241);
  padding-top: 20px;
  padding-right: 20px;
  display: contents;
  position: absolute;
  left: 0px;
  top: 0px;
}
.container {
  width: 95%;
  max-width: 1400px;
  height: 90vh;
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.5);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  border: 1px solid rgba(255, 255, 255, 0.1);
  position: absolute;
  left: 200px;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.file-list {
  width: 300px;
  background: rgba(10, 15, 40, 0.7);
  padding: 20px;
  overflow-y: auto;
  border-right: 1px solid rgba(255, 255, 255, 0.1);
  position: relative;
}

.file-list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.file-list h2 {
  color: #4fc3f7;
  padding-bottom: 10px;
  border-bottom: 1px solid rgba(79, 195, 247, 0.3);
}

.file-item {
  background: rgba(30, 40, 80, 0.6);
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid rgba(79, 195, 247, 0.2);
  position: relative;
}

.file-item:hover {
  background: rgba(40, 60, 120, 0.8);
  transform: translateY(-3px);
  border-color: rgba(79, 195, 247, 0.5);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
}

.file-item.active {
  background: rgba(30, 100, 180, 0.8);
  border-color: #4fc3f7;
}

.file-item h3 {
  color: #fff;
  font-size: 18px;
  margin-bottom: 8px;
  display: flex;
  justify-content: space-between;
}

.file-item .edit-icon {
  color: #90a4ae;
  cursor: pointer;
  padding: 2px 5px;
  border-radius: 4px;
  transition: all 0.2s;
}

.file-item .edit-icon:hover {
  background: rgba(79, 195, 247, 0.2);
  color: #4fc3f7;
}

.file-meta {
  display: flex;
  justify-content: space-between;
  color: #90a4ae;
  font-size: 14px;
  margin-top: 10px;
}

.visualization-area {
  flex: 1;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.canvas-container {
  flex: 1;
  background: rgba(0, 5, 15, 0.5);
  border-radius: 15px;
  overflow: hidden;
  position: relative;
  border: 1px solid rgba(79, 195, 247, 0.2);
  box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.5);
}

canvas {
  width: 100%;
  height: 100%;
  display: block;
}

.controls {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
  padding: 15px;
  background: rgba(10, 15, 40, 0.7);
  border-radius: 12px;
}

.info-panel {
  color: #e0e0e0;
  flex: 1;
}

.info-panel h3 {
  color: #4fc3f7;
  margin-bottom: 8px;
}

.button-group {
  display: flex;
  gap: 12px;
}

button {
  background: linear-gradient(to right, #2196f3, #21cbf3);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(33, 150, 243, 0.4);
}

button:active {
  transform: translateY(1px);
}

button.secondary {
  background: linear-gradient(to right, #6a11cb, #2575fc);
}

.stats {
  display: flex;
  gap: 20px;
  margin-top: 15px;
  flex-wrap: wrap;
}

.stat-item {
  background: rgba(30, 40, 80, 0.6);
  padding: 10px 15px;
  border-radius: 8px;
  min-width: 120px;
}

.stat-item h4 {
  color: #90a4ae;
  font-size: 14px;
  margin-bottom: 5px;
}

.stat-value {
  color: #4fc3f7;
  font-size: 18px;
  font-weight: bold;
}

.no-data {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #607d8b;
  font-size: 24px;
  text-align: center;
}

/* 模态框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: linear-gradient(135deg, #1a2a6c, #2c387e);
  border-radius: 15px;
  padding: 30px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  border: 1px solid rgba(79, 195, 247, 0.3);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.modal-header h2 {
  color: #4fc3f7;
  font-size: 24px;
}

.close-button {
  background: none;
  border: none;
  color: #90a4ae;
  font-size: 24px;
  cursor: pointer;
  transition: color 0.2s;
}

.close-button:hover {
  color: #ff5722;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  color: #a0a0ff;
  margin-bottom: 8px;
  font-weight: 500;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 12px 15px;
  border-radius: 8px;
  border: 1px solid rgba(79, 195, 247, 0.3);
  background: rgba(10, 15, 40, 0.7);
  color: white;
  font-size: 16px;
}

.form-group textarea {
  min-height: 100px;
  resize: vertical;
}

.file-upload {
  border: 2px dashed rgba(79, 195, 247, 0.5);
  border-radius: 8px;
  padding: 25px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s;
  margin-bottom: 20px;
}

.file-upload:hover {
  border-color: #4fc3f7;
  background: rgba(79, 195, 247, 0.1);
}

.file-upload p {
  color: #90a4ae;
  margin-top: 10px;
}

.file-upload i {
  font-size: 40px;
  color: #4fc3f7;
  margin-bottom: 10px;
  display: block;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.file-actions {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

.file-actions button {
  padding: 6px 12px;
  font-size: 14px;
}

.file-name {
  font-size: 12px;
  color: #4fc3f7;
  margin-top: 5px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@media (max-width: 900px) {
  .main-content {
    flex-direction: column;
  }

  .file-list {
    width: 100%;
    max-height: 200px;
  }
}
</style>
