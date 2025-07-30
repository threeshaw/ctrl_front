<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vue3 可展开列表组件</title>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css"
    />
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

      .container {
        max-width: 800px;
        width: 100%;
        margin: 0 auto;
      }

      header {
        text-align: center;
        margin-bottom: 40px;
        padding: 20px;
        background: rgba(255, 255, 255, 0.85);
        border-radius: 16px;
        box-shadow: 0 8px 32px rgba(31, 38, 135, 0.1);
        backdrop-filter: blur(4px);
        border: 1px solid rgba(255, 255, 255, 0.18);
      }

      h1 {
        color: #2c3e50;
        margin-bottom: 10px;
        font-size: 2.5rem;
      }

      .subtitle {
        color: #7f8c8d;
        font-size: 1.2rem;
        max-width: 600px;
        margin: 0 auto;
        line-height: 1.6;
      }

      .card {
        background: rgba(255, 255, 255, 0.95);
        border-radius: 16px;
        box-shadow: 0 8px 32px rgba(31, 38, 135, 0.1);
        padding: 30px;
        backdrop-filter: blur(4px);
        border: 1px solid rgba(255, 255, 255, 0.18);
        margin-bottom: 30px;
      }

      .controls {
        display: flex;
        gap: 15px;
        flex-wrap: wrap;
        margin-bottom: 25px;
      }

      button {
        background: #3498db;
        color: white;
        border: none;
        padding: 12px 20px;
        border-radius: 8px;
        cursor: pointer;
        font-weight: 600;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        gap: 8px;
      }

      button:hover {
        background: #2980b9;
        transform: translateY(-2px);
      }

      button.reset {
        background: #e74c3c;
      }

      button.reset:hover {
        background: #c0392b;
      }

      .list-container {
        display: flex;
        flex-direction: column;
        gap: 15px;
      }

      .list-item {
        background: white;
        border-radius: 12px;
        overflow: hidden;
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
        transition: all 0.3s ease;
        border: 2px solid #ecf0f1;
      }

      .list-item:hover {
        transform: translateY(-3px);
        box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1);
        border-color: #3498db;
      }

      .item-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        padding: 20px;
        cursor: pointer;
        background: #f8f9fa;
        transition: background 0.3s;
      }

      .item-header:hover {
        background: #edf2f7;
      }

      .item-title {
        font-weight: 600;
        font-size: 1.2rem;
        color: #2c3e50;
      }

      .item-icon {
        transition: transform 0.3s ease;
        color: #3498db;
      }

      .expanded .item-icon {
        transform: rotate(180deg);
      }

      .item-content {
        background: #f8f9fa;
        padding: 0 20px;
        max-height: 0;
        overflow: hidden;
        transition:
          max-height 0.5s ease,
          padding 0.5s ease;
      }

      .expanded .item-content {
        max-height: 200px;
        padding: 20px;
      }

      .options {
        display: flex;
        gap: 15px;
        margin-top: 15px;
        justify-content: center;
      }

      .option {
        width: 50px;
        height: 50px;
        display: flex;
        justify-content: center;
        align-items: center;
        background: #3498db;
        color: white;
        border-radius: 10px;
        font-weight: bold;
        font-size: 1.2rem;
        cursor: pointer;
        transition: all 0.2s ease;
        box-shadow:
          0 4px 6px rgba(50, 50, 93, 0.11),
          0 1px 3px rgba(0, 0, 0, 0.08);
      }

      .option:hover {
        transform: translateY(-3px);
        box-shadow:
          0 7px 14px rgba(50, 50, 93, 0.1),
          0 3px 6px rgba(0, 0, 0, 0.08);
        background: #2980b9;
      }

      .selected {
        background: #2ecc71;
        transform: scale(1.1);
      }

      .footer {
        text-align: center;
        margin-top: 40px;
        color: #7f8c8d;
        font-size: 0.9rem;
      }

      .highlight {
        color: #e74c3c;
        font-weight: 600;
      }

      .status-bar {
        display: flex;
        justify-content: space-between;
        background: #e3f2fd;
        padding: 12px 20px;
        border-radius: 8px;
        margin-top: 20px;
        font-size: 0.9rem;
      }

      .expanded-count {
        display: flex;
        align-items: center;
        gap: 5px;
      }

      @media (max-width: 600px) {
        .controls {
          flex-direction: column;
        }

        button {
          width: 100%;
          justify-content: center;
        }

        .options {
          flex-wrap: wrap;
        }

        .status-bar {
          flex-direction: column;
          gap: 10px;
        }
      }
    </style>
  </head>
  <body>
    <div id="app">
      <div class="container">
        <header>
          <h1>Vue3 可展开列表组件</h1>
          <p class="subtitle">
            点击列表项可展开查看选项，可以同时展开多个项目。展开后显示数字选项（1, 2,
            3），点击选项可进行选择。
          </p>
        </header>

        <main>
          <div class="card">
            <div class="controls">
              <button @click="addItem"><i class="fas fa-plus"></i> 添加新项目</button>
              <button @click="removeLastItem" :disabled="items.length <= 3">
                <i class="fas fa-minus"></i> 删除最后一项
              </button>
              <button class="reset" @click="resetItems">
                <i class="fas fa-sync"></i> 重置列表
              </button>
            </div>

            <div class="list-container">
              <div
                v-for="(item, index) in items"
                :key="item.id"
                class="list-item"
                :class="{ expanded: item.expanded }"
              >
                <div class="item-header" @click="toggleItem(index)">
                  <div class="item-title">项目 #{{ index + 1 }}: {{ item.title }}</div>
                  <div class="item-icon">
                    <i class="fas fa-chevron-down"></i>
                  </div>
                </div>

                <div class="item-content">
                  <p>请从下面的选项中选择一个数字：</p>
                  <div class="options">
                    <div
                      v-for="option in [1, 2, 3]"
                      :key="option"
                      class="option"
                      :class="{ selected: item.selected === option }"
                      @click="selectOption(item.id, option)"
                    >
                      {{ option }}
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="status-bar">
              <div>
                总项目数: <span class="highlight">{{ items.length }}</span>
              </div>
              <div class="expanded-count">
                <i class="fas fa-folder-open"></i>
                已展开项目: <span class="highlight">{{ expandedCount }}</span>
              </div>
              <div>
                已选选项: <span class="highlight">{{ selectedCount }}</span>
              </div>
            </div>
          </div>
        </main>

        <footer class="footer">
          <p>Vue3 可展开列表组件 | 点击列表项展开/收起 | 可同时展开多个项目</p>
        </footer>
      </div>
    </div>

    <script>
      const { createApp, ref, computed } = Vue

      createApp({
        setup() {
          // 初始数据
          const initialItems = [
            { id: 1, title: '前端开发', expanded: false, selected: null },
            { id: 2, title: '后端开发', expanded: false, selected: null },
            { id: 3, title: 'UI/UX 设计', expanded: false, selected: null },
            { id: 4, title: '项目测试', expanded: false, selected: null },
            { id: 5, title: '文档编写', expanded: false, selected: null },
          ]

          const items = ref([...initialItems])
          let nextId = ref(6) // 用于新项目的ID

          // 计算已选择的选项数量
          const selectedCount = computed(() => {
            return items.value.filter((item) => item.selected !== null).length
          })

          // 计算展开的项目数量
          const expandedCount = computed(() => {
            return items.value.filter((item) => item.expanded).length
          })

          // 切换列表项展开状态
          const toggleItem = (index) => {
            items.value[index].expanded = !items.value[index].expanded
          }

          // 选择选项
          const selectOption = (itemId, option) => {
            const item = items.value.find((i) => i.id === itemId)
            if (item) {
              item.selected = item.selected === option ? null : option
            }
          }

          // 添加新项目
          const addItem = () => {
            const newIndex = items.value.length + 1
            items.value.push({
              id: nextId.value++,
              title: `新项目 ${newIndex}`,
              expanded: false,
              selected: null,
            })
          }

          // 删除最后一项
          const removeLastItem = () => {
            if (items.value.length > 3) {
              items.value.pop()
            }
          }

          // 重置列表
          const resetItems = () => {
            items.value = [...initialItems]
            nextId.value = 6
          }

          return {
            items,
            selectedCount,
            expandedCount,
            toggleItem,
            selectOption,
            addItem,
            removeLastItem,
            resetItems,
          }
        },
      }).mount('#app')
    </script>
  </body>
</html>
