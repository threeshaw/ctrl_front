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
    <ul class="menu-item">
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
      <div class="listItem" @click="fun1">数据管理</div>
      <li v-for="li1 in funcList" v-if="show1" @click="handleClick($event, li1)">
        {{ li1 }}
      </li>
      <li class="listItem">模型辨识</li>
      <li class="listItem">参数组态</li>
      <li class="listItem">仿真测试</li>
      {{ value }}
    </div>
    <div class="container">
      <!-- 侧边栏 -->
      <div class="blocker">文件列表</div>
      <!-- 编辑器区域 -->
      <div class="editor-container" v-if="showData">数据展示</div>
      <div class="ioChart" v-if="false">输入输出设置</div>
      <div class="modelChoose" v-if="false">模型选择</div>
      <div class="resShow" v-if="false">结果展示</div>
    </div>
  </body>
</template>
<script setup>
import leftlist from './leftlist.vue'
import { ref } from 'vue'

const funcList = ref(['file1', 'file2'])
const show1 = ref(false)
const showData = ref(false)
const fun1 = () => {
  show1.value = !show1.value
  console.log(show1.value)
}
const handleClick = (event, item) => {
  event.preventDefault() // 阻止默认行为
  console.log('DOM event:', event)
  showData.value = true
  console.log('Item data:', item)
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
  display: inline-block;
  border: 2px blue solid;
  border-radius: 0px;
  position: relative;
  left: 0px;
  width: 2800px;
  z-index: 1;
  overflow: scroll;
}
.leftCon {
  display: inline;
  width: 100px;
  height: 200px;
  overflow: hidden;
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
