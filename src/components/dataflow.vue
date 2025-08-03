<template>
  <body>
    <div class="container1">
      <div class="content">
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th style="width: 5%">ID</th>
                <th style="width: 35%">数据名称</th>
                <th style="width: 30%">输入数据</th>
                <th style="width: 30%">输出数据</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in data" :key="item.id" :class="getRowClass(item)">
                <td>{{ item.id }}</td>
                <td>{{ item.name }}</td>
                <td>
                  <div class="checkbox-group">
                    <input
                      type="checkbox"
                      :id="'input-' + item.id"
                      v-model="item.isInput"
                      @change="updateStatus(item, 'input', $event)"
                    />
                    <label :for="'input-' + item.id">输入数据</label>
                  </div>
                </td>
                <td>
                  <div class="checkbox-group">
                    <input
                      type="checkbox"
                      :id="'output-' + item.id"
                      v-model="item.isOutput"
                      @change="updateStatus(item, 'output', $event)"
                    />
                    <label :for="'output-' + item.id">输出数据</label>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="summary">
          <div class="summary-card">
            <h3>总数据项</h3>
            <div class="value">{{ data.length }}</div>
          </div>
          <div class="summary-card">
            <h3>输入数据</h3>
            <div class="value input-count">{{ inputCount }}</div>
          </div>
          <div class="summary-card">
            <h3>输出数据</h3>
            <div class="value output-count">{{ outputCount }}</div>
          </div>
          <div class="summary-card">
            <h3>未设置</h3>
            <div class="value empty-count">{{ emptyCount }}</div>
          </div>
        </div>
      </div>
      <div class="container2">
        <div class="dropdown-container">
          <input
            type="text"
            class="select-input"
            placeholder="请选择模型"
            v-model="selectedOption"
            @click="toggleDropdown"
            readonly
          />
          <div class="options-container" :class="{ active: isOpen }">
            <div
              v-for="(option, index) in options"
              :key="index"
              class="option"
              :class="{ selected: option.value === selectedOption }"
              @click="selectOption(option)"
            >
              <span>{{ option.label }}</span>
            </div>
          </div>
        </div>

        <div class="selected-value" v-if="selectedOption">
          当前选择：<span>{{ selectedOptionLabel }}</span>
        </div>
      </div>
    </div>
  </body>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const initialData = [
  { id: 1, name: '用户配置文件', isInput: false, isOutput: false },
  { id: 2, name: '系统日志', isInput: false, isOutput: false },
  { id: 3, name: '传感器读数', isInput: false, isOutput: false },
  { id: 4, name: '数据库备份', isInput: false, isOutput: false },
  { id: 5, name: 'API响应数据', isInput: false, isOutput: false },
]

const data = ref([...initialData])
let nextId = initialData.length + 1

// 计算属性
const inputCount = computed(() => {
  return data.value.filter((item) => item.isInput && !item.isOutput).length
})

const outputCount = computed(() => {
  return data.value.filter((item) => !item.isInput && item.isOutput).length
})

const emptyCount = computed(() => {
  return data.value.filter((item) => !item.isInput && !item.isOutput).length
})

// 方法
const addDataItem = () => {
  const newItem = {
    id: nextId++,
    name: `数据项 ${nextId}`,
    isInput: false,
    isOutput: false,
  }
  data.value.push(newItem)
}

const resetData = () => {
  data.value = [...initialData]
  nextId = initialData.length + 1
  console.log(initialData[1].name)
}

const updateStatus = (item, type, event) => {
  if (type === 'input' && event.target.checked) {
    item.isOutput = false
  } else if (type === 'output' && event.target.checked) {
    item.isInput = false
  }
}

const getRowClass = (item) => {
  if (item.isInput) return 'input-data'
  if (item.isOutput) return 'output-data'
  return 'empty-data'
}

const isOpen = ref(false)
const selectedOption = ref('')
const options = ref([
  { value: 'javascript', label: 'JavaScript', icon: '🟨' },
  { value: 'python', label: 'Python', icon: '🐍' },
  { value: 'java', label: 'Java', icon: '☕' },
  { value: 'cplusplus', label: 'C++', icon: '🔷' },
  { value: 'go', label: 'Go', icon: '🚀' },
])

const selectedOptionLabel = computed(() => {
  if (!selectedOption.value) return ''
  const option = options.value.find((opt) => opt.value === selectedOption.value)
  return option ? option.label : ''
})

// 切换下拉框状态
const toggleDropdown = () => {
  isOpen.value = !isOpen.value
}

// 选择选项
const selectOption = (option) => {
  selectedOption.value = option.value
  isOpen.value = false
}

// 关闭下拉框
const closeDropdown = () => {
  isOpen.value = false
}

// 处理键盘事件
const handleKeydown = (e) => {
  if (!isOpen.value) return

  switch (e.key) {
    case 'Escape':
      closeDropdown()
      break
  }
}

// 点击外部关闭下拉框
const handleClickOutside = (event) => {
  const dropdownContainer = document.querySelector('.dropdown-container')
  if (dropdownContainer && !dropdownContainer.contains(event.target)) {
    closeDropdown()
  }
}

// 生命周期钩子
onMounted(() => {
  document.addEventListener('click', handleClickOutside)
  document.addEventListener('keydown', handleKeydown)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside)
  document.removeEventListener('keydown', handleKeydown)
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
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.container1 {
  width: 100%;
  max-width: 1200px;
  min-width: 1000px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  overflow: scroll;
  display: block;
}

.content {
  padding: 25px;
}

.controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

button {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
}

button.add {
  background: linear-gradient(90deg, #2ecc71, #27ae60);
  color: white;
}

button.reset {
  background: linear-gradient(90deg, #e74c3c, #c0392b);
  color: white;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

button:active {
  transform: translateY(1px);
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
}

th {
  background: #3498db;
  color: white;
  padding: 15px;
  text-align: left;
  font-weight: 600;
}

tr:nth-child(even) {
  background-color: #f8f9fa;
}

tr:hover {
  background-color: #e8f4fc;
}

td {
  padding: 15px;
  border-bottom: 1px solid #e0e6ed;
}

.checkbox-container {
  display: flex;
  gap: 15px;
}

.checkbox-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.checkbox-group input {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

.checkbox-group label {
  cursor: pointer;
  user-select: none;
}

.input-data .status {
  color: #27ae60;
  font-weight: 600;
}

.output-data .status {
  color: #e67e22;
  font-weight: 600;
}

.empty-data .status {
  color: #95a5a6;
}

.actions {
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

.actions button {
  padding: 8px 15px;
  background: #95a5a6;
  color: white;
}

.summary {
  margin-top: 25px;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 10px;
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 15px;
}

.summary-card {
  background: white;
  padding: 15px 20px;
  border-radius: 10px;
  text-align: center;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.08);
  min-width: 150px;
}

.summary-card h3 {
  color: #7f8c8d;
  font-size: 1rem;
  margin-bottom: 10px;
}

.summary-card .value {
  font-size: 2rem;
  font-weight: 700;
}

.input-count {
  color: #27ae60;
}

.output-count {
  color: #e67e22;
}

.empty-count {
  color: #95a5a6;
}

.dropdown-container {
  position: relative;
  margin: 15px 0;
}

.select-input {
  width: 100%;
  padding: 14px 20px;
  font-size: 1rem;
  border: 2px solid #e0e6ed;
  border-radius: 10px;
  background-color: #f8f9fa;
  cursor: pointer;
  outline: none;
  transition: all 0.3s ease;
}

.select-input:focus {
  border-color: #3498db;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
}

.select-input::placeholder {
  color: #95a5a6;
}

.options-container {
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.1);
  margin-top: 8px;
  overflow: hidden;
  opacity: 0;
  transform: translateY(-10px);
  visibility: hidden;
  transition: all 0.3s ease;
  z-index: 100;
  max-height: 300px;
  overflow-y: auto;
}

.options-container.active {
  opacity: 1;
  transform: translateY(0);
  visibility: visible;
}

.option {
  padding: 14px 20px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  border-bottom: 1px solid #f1f5f9;
}

.option:last-child {
  border-bottom: none;
}

.option:hover {
  background-color: #f1f8ff;
}

.option .icon {
  margin-right: 12px;
  font-size: 1.2rem;
  width: 24px;
  text-align: center;
}

.option.selected {
  background-color: #e1f0fa;
  font-weight: 600;
}

.selected-value {
  margin-top: 20px;
  padding: 15px;
  background-color: #f8f9fa;
  border-radius: 8px;
  text-align: center;
  font-size: 1.1rem;
}

.selected-value span {
  color: #3498db;
  font-weight: 600;
}
</style>
