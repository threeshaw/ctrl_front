<template>
  <div class="vscode-top-bar">
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
    <div class="click-listf" id="clickfile">
      <div class="list-item">新建</div>
      <div class="list-item">保存</div>
      <div class="list-item">打开</div>
      <div class="list-item">另存为</div>
    </div>
  </div>
  <div class="container">
    fff
    <!-- 侧边栏 -->
    <div class="blocker">leftlist</div>
    <!-- 编辑器区域 -->
    <div class="editor-container">
      <!-- 点击时显示的列表 -->
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import * as XLSX from 'xlsx'
import leftlist from './leftlist.vue'
</script>
<style>
.container {
  display: block;
  border: 2px blue solid;
}

.menu-item {
  display: inline;
}

.blocker {
  width: 100px;
  position: relative;
  background-color: yellow;
  height: auto;
  min-height: 200px;
  left: 20px;
}
.menu-bar {
  width: 1000px;
  height: 50px;
  display: inline;
  background-color: brown;
  position: absolute;
  left: 0px;
  top: 0px;
}

.click-listf {
  border: 2px;
  border-style: dashed;
  border-color: black;
  position: absolute;
  left: 0px;
  top: 50px;
  z-index: 10;
}
</style>
