<template>
  <body>
    <div class="vscode-top-bar">
      <div class="menu-bar">
        <ul class="menu-item" @click="showlist">
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
      <div class="click-listf" id="clickfile" v-show="dispfile" @click="triggerFileInput">
        <div class="list-item">新建</div>
        <div class="list-item">保存</div>
        <div class="list-item">打开</div>
        <div class="list-item">另存为</div>
      </div>
      <!-- <div class="window-title">index.html</div> -->
      <!-- <div class="window-controls">
        <div class="window-control minimize"></div>
        <div class="window-control maximize"></div>
        <div class="window-control close"></div>
      </div> -->
    </div>
    <div class="blocker">leftlist</div>
    <!-- <div class="modellist">模型选择</div> -->
    <div class="container">
      <!-- 侧边栏 -->
      <div class="sidebar">
        <div>
          <ul>
            数据输入
          </ul>
          <input
            type="file"
            ref="fileInput"
            @change="handleFileChange"
            accept=".xlsx, .xls ,.csv"
            style="display: none"
          />
        </div>

        <div class="datalist" v-show="false">
          <ul class="input1" v-for="item in datalist">
            {{
              item
            }}
          </ul>
        </div>
      </div>

      <!-- 编辑器区域 -->
      <div class="editor-container">
        <div class="tabs"></div>
        <canvas class="mainCanvas"></canvas>
        <!-- 点击时显示的列表 -->
        <!-- <div class="click-list" id="click-list">
          <div class="list-item"><span class="icon">📄</span> 新建文件</div>
          <div class="list-item"><span class="icon">📁</span> 新建文件夹</div>
          <div class="list-item"><span class="icon">🔍</span> 查找</div>
          <div class="list-item"><span class="icon">✂️</span> 剪切</div>
          <div class="list-item"><span class="icon">📋</span> 复制</div>
          <div class="list-item"><span class="icon">📋</span> 粘贴</div>
          <div class="list-item"><span class="icon">🔄</span> 重新加载</div>
          <div class="list-item"><span class="icon">⚙️</span> 设置</div>
        </div> -->
      </div>
    </div>

    <!-- 底部控制台 -->
    <div class="console-container">
      <div class="console-header">
        <div class="tab">输出</div>
      </div>
      <div class="console-content"></div>
    </div>

    <!-- 状态栏 
    <div class="status-bar">
      <div class="status-item">
        <span>main</span>
      </div>
      <div class="status-item">
        <span>UTF-8</span>
      </div>
      <div class="status-item">
        <span>HTML</span>
      </div>
      <div class="status-item">
        <span>行 24, 列 12</span>
      </div>
      <div class="status-item">
        <span>空格: 4</span>
      </div>
    </div>-->
  </body>
</template>
<script setup>
import { ref, computed } from 'vue'
import * as XLSX from 'xlsx'
import leftlist from './leftlist.vue'
let dispfile = ref(false)
const datalist = ref([])
const fileInput = ref(null)

const showlist = () => {
  dispfile.value = !dispfile.value
  console.log(dispfile.value)
}

const triggerFileInput = () => {
  fileInput.value.click()
}

// 处理文件选择
const handleFileChange = (e) => {
  const file = e.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (event) => {
    try {
      const data = new Uint8Array(event.target.result)
      const workbook = XLSX.read(data, { type: 'array' })

      // 获取第一个工作表
      const firstSheetName = workbook.SheetNames[0]
      const worksheet = workbook.Sheets[firstSheetName]
      datalist.value.push('输入' + firstSheetName)
      // 转换为JSON
      const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 })
      console.log(datalist)
      // 处理数据：取前5行5列
    } catch (error) {
      console.error('Excel解析错误:', error)
      alert('文件解析失败，请检查文件格式')
    }
  }
  reader.readAsArrayBuffer(file)
}

const initialItems = [
  { title: '前端开发', expanded: false, selected: null },
  { title: '后端开发', expanded: false, selected: null },
  { title: 'UI/UX 设计', expanded: false, selected: null },
  { title: '项目测试', expanded: false, selected: null },
  { title: '文档编写', expanded: false, selected: null },
]

const items = ref([...initialItems])
const expandedItem = ref(null)

// 计算已选择的选项数量
const selectedCount = computed(() => {
  return items.value.filter((item) => item.selected !== null).length
})

// 切换列表项展开状态
const toggleItem = (index) => {
  if (expandedItem.value === index) {
    // 如果点击的是已展开的项，则收起
    items.value[index].expanded = false
    expandedItem.value = null
  } else {
    // 收起之前展开的项
    if (expandedItem.value !== null) {
      items.value[expandedItem.value].expanded = false
    }

    // 展开当前项
    items.value[index].expanded = true
    expandedItem.value = index
  }
}
</script>
<style>
.blocker {
  width: 100px;
  background-color: yellow;
  height: auto;
  min-height: 200px;
  left: 20px;
}
.menu-bar {
  width: 1000px;
  height: 50px;
  display: flex;
  background-color: brown;
  position: absolute;
  left: 0px;
  top: 0px;
}
.container {
  position: absolute;
  left: 0px;
  top: 100px;
  background-color: rgb(40, 23, 2);
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
