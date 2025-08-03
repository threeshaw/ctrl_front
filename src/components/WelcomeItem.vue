<template>
  <div class="container">
    <div class="dropdown-container">
      <input
        type="text"
        class="select-input"
        placeholder="请选择您喜欢的编程语言"
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
</template>
<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
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
<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

body {
  background: linear-gradient(135deg, #f5f7fa 0%, #e4edf5 100%);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

#app {
  width: 100%;
  max-width: 600px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  padding: 30px;
}

.container {
  display: flex;
  flex-direction: column;
  gap: 25px;
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
