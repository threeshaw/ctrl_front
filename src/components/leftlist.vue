<template>
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
</template>

<script setup>
import { ref, computed } from 'vue'
const show1 = ref(false)
const show2 = ref(false)
const show3 = ref(false)
const changeState1 = () => {
  show1.value = !show1.value
}
const changeState2 = () => {
  show2.value = !show2.value
}
const changeState3 = () => {
  show3.value = !show3.value
}
const list1 = ['f1', 'f2', 'f3']
const list2 = []
const list3 = []
// 初始数据
</script>

<style>
.containerleft {
  max-width: 500px;
  width: 200px;
  height: auto;
  background-color: rgb(20, 9, 241);
  padding-top: 20px;
  padding-right: 20px;
  text-align: center;
  display: contents;
  position: absolute;
  left: 0px;
  top: 0px;
}
</style>
