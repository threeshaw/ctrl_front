<template>
  <div class="vscode-top-bar">
    <div class="menu-bar">
      <div class="menu-item">文件</div>
      <div class="menu-item">编辑</div>
      <div class="menu-item">选择</div>
      <div class="menu-item">查看</div>
      <div class="menu-item">运行</div>
      <div class="menu-item">终端</div>
      <div class="menu-item">帮助</div>
    </div>
    <div class="window-title">index.html</div>
    <div class="window-controls">
      <div class="window-control minimize"></div>
      <div class="window-control maximize"></div>
      <div class="window-control close"></div>
    </div>
  </div>

  <div class="container">
    <!-- 侧边栏 -->
    <div class="sidebar">
      <p>数据输入</p>
      <div class="datalist">
        <ul class="input1"></ul>
        <ul class="input2"></ul>
      </div>
      <div class="modellist">模型选择</div>
    </div>

    <!-- 编辑器区域 -->
    <div class="editor-container">
      <div class="tabs"></div>

      <!-- 点击时显示的列表 -->
      <div class="click-list" id="click-list">
        <div class="list-item"><span class="icon">📄</span> 新建文件</div>
        <div class="list-item"><span class="icon">📁</span> 新建文件夹</div>
        <div class="list-item"><span class="icon">🔍</span> 查找</div>
        <div class="list-item"><span class="icon">✂️</span> 剪切</div>
        <div class="list-item"><span class="icon">📋</span> 复制</div>
        <div class="list-item"><span class="icon">📋</span> 粘贴</div>
        <div class="list-item"><span class="icon">🔄</span> 重新加载</div>
        <div class="list-item"><span class="icon">⚙️</span> 设置</div>
      </div>
    </div>
  </div>

  <!-- 底部控制台 -->
  <div class="console-container">
    <div class="console-header">
      <div class="tab active">问题</div>
      <div class="tab">输出</div>
      <div class="tab">调试控制台</div>
      <div class="tab">终端</div>
    </div>
    <div class="console-content"></div>
  </div>

  <!-- 状态栏 -->
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
  </div>
</template>
<script></script>
<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Segoe UI', 'Microsoft YaHei', sans-serif;
}

body {
  background-color: #1e1e1e;
  color: #d4d4d4;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* VS Code顶部控制台样式 */
.vscode-top-bar {
  background-color: #2d2d2d;
  height: 40px;
  display: flex;
  align-items: center;
  padding: 0 15px;
  border-bottom: 1px solid #3c3c3c;
  -webkit-app-region: drag;
}

.menu-bar {
  display: flex;
  gap: 20px;
  margin-right: auto;
}

.menu-item {
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
  font-size: 13px;
  -webkit-app-region: no-drag;
}

.menu-item:hover {
  background-color: #3a3d41;
}

.window-controls {
  display: flex;
  gap: 12px;
  -webkit-app-region: no-drag;
}

.window-control {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  cursor: pointer;
}

.minimize {
  background-color: #f1fa8c;
}

.maximize {
  background-color: #50fa7b;
}

.close {
  background-color: #ff5555;
}

/* 主内容区域 */
.container {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* 侧边栏样式 */
.sidebar {
  width: 200px;
  background-color: #333333;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 15px;
  border-right: 1px solid #252526;
}

.datalist {
  width: 180px;
  height: 200px;
  background-color: #f5e505;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 15px;
  border-right: 1px solid #252526;
}
.modellist {
  margin-top: 20px;
  width: 190px;
  height: 240px;
  background-color: #f5e505;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 15px;
}
/* 编辑器区域 */
.editor-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.tabs {
  display: flex;
  background-color: #252526;
  border-bottom: 1px solid #2d2d2d;
  height: 35px;
}

.tab {
  padding: 8px 15px;
  font-size: 13px;
  background-color: #2d2d2d;
  border-right: 1px solid #2d2d2d;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.tab.active {
  background-color: #1e1e1e;
  color: #ffffff;
}

.tab .close-icon {
  margin-left: 10px;
  opacity: 0.5;
}

.tab .close-icon:hover {
  opacity: 1;
}

.editor {
  flex: 1;
  padding: 15px;
  overflow-y: auto;
  font-family: 'Consolas', 'Courier New', monospace;
  font-size: 14px;
  line-height: 1.5;
  position: relative;
  cursor: text;
}

.editor-line {
  margin-bottom: 2px;
}

.comment {
  color: #6a9955;
}

.keyword {
  color: #569cd6;
}

.function {
  color: #dcdcaa;
}

.string {
  color: #ce9178;
}

.number {
  color: #b5cea8;
}

/* 点击时显示的列表 */
.click-list {
  position: absolute;
  background-color: #252526;
  border: 1px solid #454545;
  border-radius: 4px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  z-index: 100;
  display: none;
  width: 250px;
  max-height: 300px;
  overflow-y: auto;
}

.click-list.visible {
  display: block;
}

.list-item {
  padding: 8px 15px;
  font-size: 13px;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.list-item:hover {
  background-color: #2a2d2e;
}

.list-item .icon {
  margin-right: 10px;
  color: #569cd6;
}

/* 状态栏 */
.status-bar {
  height: 22px;
  background-color: #0078d4;
  display: flex;
  align-items: center;
  padding: 0 10px;
  font-size: 12px;
  justify-content: space-between;
  color: #ffffff;
}

.status-item {
  display: flex;
  align-items: center;
}

.status-item i {
  margin-right: 5px;
}

/* 底部控制台 */
.console-container {
  height: 200px;
  background-color: #1e1e1e;
  border-top: 1px solid #3c3c3c;
  display: flex;
  flex-direction: column;
}

.console-header {
  height: 35px;
  background-color: #252526;
  display: flex;
  align-items: center;
  padding: 0 15px;
  font-size: 13px;
  border-bottom: 1px solid #2d2d2d;
}

.console-header .tab {
  background-color: transparent;
  border-bottom: 1px solid transparent;
}

.console-header .tab.active {
  border-bottom: 1px solid #0078d4;
}

.console-content {
  flex: 1;
  padding: 10px 15px;
  overflow-y: auto;
  font-family: 'Consolas', 'Courier New', monospace;
  font-size: 13px;
  line-height: 1.4;
}

.console-line {
  margin-bottom: 5px;
}

.console-line.error {
  color: #f48771;
}

.console-line.warning {
  color: #dcdcaa;
}

.console-line.info {
  color: #9cdcfe;
}

.console-line.success {
  color: #50fa7b;
}
ul.input1 {
  list-style-type: square;
}
</style>
