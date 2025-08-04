<template>
  <div class="color-box" :style="{ backgroundColor: currentColor }" @click="changeColor"></div>
</template>

<script setup>
import { ref } from 'vue'

// 预设颜色数组
const colors = ['#3498db', '#2ecc71', '#e74c3c', '#f39c12', '#9b59b6', '#1abc9c']
const currentColor = ref(colors[0]) // 初始颜色

// 点击切换颜色
const changeColor = () => {
  const currentIndex = colors.indexOf(currentColor.value)
  const nextIndex = (currentIndex + 1) % colors.length
  currentColor.value = colors[nextIndex]
}
</script>

<style scoped>
.color-box {
  width: 150px;
  height: 150px;
  border: 2px solid #2c3e50;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
</style>
