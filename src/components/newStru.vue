<template>
  <div class="menu-bar">
    <ul class="menu-item">
      预览
    </ul>
    <ul class="menu-item">
      趋势
    </ul>
    <ul class="menu-item">
      新建
    </ul>
    <ul class="menu-item">
      删除
    </ul>
    <ul class="menu-item">
      分析
    </ul>
    <ul class="menu-item" @click="fileIn">
      导入
    </ul>
    <ul class="menu-item">
      导出
    </ul>
  </div>
  <div class="click-listf" id="clickfile" v-if="false">
    <div class="list-item">新建</div>
    <div class="list-item">保存</div>
    <div class="list-item">打开</div>
    <div class="list-item">另存为</div>
  </div>
  <body>
    <div class="leftCon">
      <div>
        <div @click="fun1">数据管理</div>
        <li v-for="li1 in fileList" v-if="show1" @click="handleClick1($event, li1)">
          {{ li1 }}
        </li>
      </div>

      <div>
        <div @click="fun2">模型辨识</div>
        <li v-for="li1 in modelIden" v-if="show2" @click="handleClick2($event, li1)">
          {{ li1 }}
        </li>
      </div>

      <li class="listItem">参数组态</li>
      <li class="listItem">仿真测试</li>
      {{ value }}
    </div>
    <div class="container">
      <!-- 侧边栏 -->
      <dataList ref="datas"></dataList>
    </div>
  </body>
</template>
<script setup>
import { ref } from 'vue'
import dataList from './datalist.vue'

const show1 = ref(false)
const show2 = ref(false)
const showData = ref(false)
const showModel = ref(false)
const modelIden = ref(['过程变量', '过程曲线', '过程模型', '模型输出'])
const fileList = ref([])
const datas = ref()
const fun1 = () => {
  show1.value = !show1.value
  fileList.value = datas.value.leftFileList
  console.log(show1.value)
}
const fun2 = () => {
  show2.value = !show2.value
}
const handleClick1 = (event, item) => {
  event.preventDefault() // 阻止默认行为

  showData.value = true
  showModel.value = false
}

const handleClick2 = (event, item) => {
  event.preventDefault() // 阻止默认行为

  showData.value = false
  showModel.value = true
}

function fileIn() {
  datas.value.openNewFileModal()
}
</script>

<style scoped>
.menu-bar {
  width: 1000px;
  height: 50px;
  display: fi;
  background-color: brown;
  position: fixed;
  left: 0px;
  top: 0px;
}
.menu-item {
  display: inline;
}
.blocker {
  width: 200px;
  overflow: scroll;
  height: auto;
  background-color: aqua;
}
.container {
  height: 1000px;
  display: inline-block;
  border: 2px blue solid;
  border-radius: 0px;
  position: relative;
  left: 50px;
  width: 2800px;
  z-index: 5;
  overflow: scroll;
}
.leftCon {
  display: inline;
  width: 150px;
  height: 400px;
  overflow: scroll;
  position: absolute;
  left: 0px;
  top: 80px;
  z-index: 2;
  border: 2px blue solid;
  border-radius: 2px;
}
.listItem {
  display: flex;
  font-size: medium;
}
</style>
