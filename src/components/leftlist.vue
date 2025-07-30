<template>
  <div class="container">
    <ul class="mainlist">
      <div @click="changeState">file</div>
      <div v-for="item in filelists" v-if="showfile">
        {{ item }}
      </div>
    </ul>
    <ul class="mainlist">
      car
      <li class="file">f1</li>
      <li class="file">f2</li>
      <li class="file">f3</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
const showfile = ref(true)
const changeState = () => {
  showfile.value = !showfile.value
  console.log(showfile.value)
}
const filelists = ['f1', 'f2', 'f3']
// 初始数据
</script>

<style>
.container {
  max-width: 100px;
  width: 300px;
  height: auto;
  background-color: aqua;
  padding-top: 5%;
}
</style>
