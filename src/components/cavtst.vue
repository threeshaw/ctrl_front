<template>
    <div class="content">
      <canvas ref="myCanvas" height="400" width="450" 
      @mousedown="onDown" @mouseup="onUp" @mousemove="onMove" @mouseleave="onLeave">
      </canvas>

    </div>
</template>
 
<script>

export default {
  name: "canvasTest",
  mounted() {
    this.drawCanvas()
  },
  data(){
    blockNum:0
    ischosen : false
    blocklist :[]
    blockmid :[100,200]
  },

  methods: {
    drawCanvas() {
      const canvas = this.$refs.myCanvas //获取 canvas 组件
      const ctx = canvas.getContext('2d') //获取 canvas 绘图上下文
      console.log('1')
        ctx.fillRect( 10, 10, canvas.width, canvas.height)  //绘制背景图片
        // 设置线条样式为圆滑转折点和端点
        ctx.lineJoin = 'round'; // 设置转折点样式为圆滑
        ctx.lineCap = 'round'; // 设置端点样式为圆滑
        // 设置线条宽度
        ctx.lineWidth = 4; // 设置线条宽度为5像素
        ctx.moveTo(0,0); 	// 路径绘制命令的起点
        ctx.lineTo(350,50); // 路径绘制转折点
        ctx.stroke();

        ctx.moveTo(50,200);
        ctx.lineTo(250,200);
        ctx.lineTo(250,350);
        ctx.lineTo(50,350);
        ctx.lineTo(50,200);
        ctx.strokeStyle = 'rgb(250,46,150)'
        ctx.stroke();
      },
      onMove() {
       document.addEventListener('mousemove', function(event) {
       let mouseX = event.clientX; // 鼠标相对于浏览器窗口可视区域的X坐标
       let mouseY = event.clientY; // 鼠标相对于浏览器窗口可视区域的Y坐标

    console.log('Mouse X: ' + mouseX + ', Mouse Y: ' + mouseY);
});
      }
    }
  }
</script>