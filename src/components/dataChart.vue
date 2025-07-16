<template>
  <div class="container">
    <!-- 文件选择按钮 -->
    <button type="primary" @click="triggerFileInput">
      选择Excel文件
    </button>
    <input 
      type="file" 
      ref="fileInput" 
      accept=".xlsx, .xls" 
      @change="handleFileChange"
      hidden
    >
    
    <!-- 图表容器 -->
    <div v-if="chartVisible" class="chart-container">
      <div ref="chart" style="width: 100%; height: 400px;"></div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import * as XLSX from 'xlsx';
import * as echarts from 'echarts';

export default {
  setup() {
    // 响应式数据
    const fileInput = ref(null);
    const chart = ref(null);
    const chartVisible = ref(false);
    const excelData = ref([]);
    
    // 初始化图表
    let myChart = null;
    onMounted(() => {
      if (chart.value) {
        myChart = echarts.init(chart.value);
      }
    });

    // 触发文件选择
    const triggerFileInput = () => {
      fileInput.value.click();
    };

    // 处理文件选择
    const handleFileChange = (e) => {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const data = new Uint8Array(event.target.result);
          const workbook = XLSX.read(data, { type: 'array' });
          
          // 获取第一个工作表
          const firstSheetName = workbook.SheetNames[0];
          const worksheet = workbook.Sheets[firstSheetName];
          
          // 转换为JSON
          const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
          
          // 获取第一行数据（行向量）
          if (jsonData.length > 0) {
            excelData.value = jsonData[0].filter(val => typeof val === 'number');
            renderChart();
            chartVisible.value = true;
          }
        } catch (error) {
          console.error('文件解析错误:', error);
          alert('文件解析失败，请检查文件格式');
        }
      };
      reader.readAsArrayBuffer(file);
    };

    // 渲染图表
    const renderChart = () => {
      if (!myChart) return;
      
      const option = {
        title: {
          text: 'Excel数据趋势图',
          left: 'center'
        },
        tooltip: {
          trigger: 'axis'
        },
        xAxis: {
          type: 'category',
          data: excelData.value.map((_, index) => `点${index + 1}`),
          name: '数据点'
        },
        yAxis: {
          type: 'value',
          name: '数值'
        },
        series: [{
          name: '行向量数据',
          type: 'line',
          data: excelData.value,
          smooth: true,
          symbol: 'circle',
          symbolSize: 8,
          lineStyle: {
            width: 3,
            color: '#5470C6'
          },
          itemStyle: {
            color: '#91CC75'
          },
          areaStyle: {
            color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [
              { offset: 0, color: 'rgba(84, 112, 198, 0.5)' },
              { offset: 1, color: 'rgba(84, 112, 198, 0.1)' }
            ])
          }
        }],
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        }
      };

      myChart.setOption(option);
      window.addEventListener('resize', () => myChart.resize());
    };

    return {
      fileInput,
      chart,
      chartVisible,
      triggerFileInput,
      handleFileChange
    };
  }
}
</script>

<style>
.container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 20px;
  text-align: center;
}

.chart-container {
  margin-top: 30px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.1);
  padding: 20px;
}
</style>