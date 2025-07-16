<template>
    <div>
        <canvas id="canvas" style="background-color: #94e8ea"></canvas>
    </div>
</template>

<script>
    // 获取canvas
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');

    // 所有绘制的图形推入数组中。
    let shapeCollection = []

    // 设置宽高
    const w = 1200, h = 800;
    canvas.width = w * devicePixelRatio;
    canvas.height = h * devicePixelRatio;
    class RectFactory {
        /**
         * @startX：鼠标 点击时 的坐标轴 X 值
         * @startY：鼠标 点击时 的坐标轴 Y 值
         * @endX：鼠标 抬起时 的坐标轴 X 值
         * @endY：鼠标 抬起时 的坐标轴 Y 值
         * */
        constructor(startX, startY, endX, endY) {
            this.startX = startX;
            this.startY = startY;
            this.endX = endX;
            this.endY = endY;

        }
        // 为了保持不同的鼠标绘制方向都能绘制出图形，对坐标进行处理。
        get minX() {
            return Math.min(this.startX, this.endX);
        }
        get maxX() {
            return Math.max(this.startX, this.endX);
        }
        get minY() {
            return Math.min(this.startY, this.endY);
        }
        get maxY() {
            return Math.max(this.startY, this.endY);
        }

        draw() {
            ctx.beginPath();
            ctx.moveTo(this.minX * devicePixelRatio, this.minY * devicePixelRatio);
            ctx.lineTo(this.maxX * devicePixelRatio, this.minY * devicePixelRatio);
            ctx.lineTo(this.maxX * devicePixelRatio, this.maxY * devicePixelRatio);
            ctx.lineTo(this.minX * devicePixelRatio, this.maxY * devicePixelRatio);
            ctx.lineTo(this.minX * devicePixelRatio, this.minY * devicePixelRatio);
            ctx.fillStyle = '#ffb60f';
            ctx.fill();
            ctx.strokeStyle = '#fff';
            ctx.lineCap = 'square';
            ctx.lineWidth = 1 * devicePixelRatio;
            ctx.stroke();
        }

        // 判断当前点击位置是否在图形内部
        isInside(x, y) {
            return x >= this.minX && x <= this.maxX && y >= this.minY && y <= this.maxY
        }
    }

    canvas.onmousedown = (e) => {
        const rect = canvas.getBoundingClientRect();
        const clickX = e.clientX - rect.left;
        const clickY = e.clientY - rect.top;
        const shape = getRect(clickX, clickY)

        if (shape) {
            moveRect(e, clickX, clickY, rect, shape)
        } else {
            drawRect(e, clickX, clickY, rect)
        }

    }
    canvas.onmouseup = () => {
        canvas.onmousemove = null;
        window.onmousemove = null;
    };

    // 鼠标点击canvas查看是否点击到了已经绘制的路线，若是，则返回相关线的对象，若否，返回null
    const getRect = (x, y) => {
        for (let i = shapeCollection.length - 1; i >= 0; i--) {
            const element = shapeCollection[i];
            if (element.isInside(x, y)) {
                return element;
            }
        }
        return null
    }

    const drawRect = (e, clickX, clickY, rect) => {
        const shape = new RectFactory(clickX, clickY)
        shapeCollection.push(shape)
        window.onmousemove = (evt) => {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            shape.endX = evt.clientX - rect.left;
            shape.endY = evt.clientY - rect.top;
        }
    }

    const moveRect = (e, clickX, clickY, rect, shape) => {
        const { startX, startY, endX, endY } = shape;
        window.onmousemove = (evt) => {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            const distanceX = evt.clientX - rect.left - clickX;
            const distanceY = evt.clientY - rect.top - clickY;
            shape.startX = startX + distanceX;
            shape.startY = startY + distanceY;
            shape.endX = endX + distanceX;
            shape.endY = endY + distanceY;
        }
    };

    const draw = () => {
        requestAnimationFrame(draw)
        for (const pp of shapeCollection) {
            pp.draw()
        }
    }
    draw()
</script>