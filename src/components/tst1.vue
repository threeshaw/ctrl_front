<template>
    <div ref="containerRef" class="common-layout">
        <canvas ref="canvasRef" @mousedown="onDown" @mouseup="onUp" @mousemove="onMove" @mouseleave="onLeave"
            :style="{ width: cWidth + 'px', height: cHeight + 'px' }" class="canvas_css"></canvas>
    </div>
</template>
<script setup>
import { ref, onMounted, onUnmounted, reactive } from 'vue'
let cWidth = 800
let cHeight = 800

// 拖拽状态
const isDragging = ref();
const dragTarget = ref(null); // 拖拽的目标元素
// 是否是拖拽的整体
const isDraggingOverall = ref(false); // 是否是拖拽的整体
let dragRectDots = reactive([]); // 拖拽的端点集合
let fixedPoint = reactive({}); // 固定的端点集合
const containerRef = ref(null);
const canvasRef = ref(null)
let canvas = null
let rect = null
let ctx = null
let offsetX = ref(0), offsetY = ref(0);
const radius = 5; // 端点圆的半径
const elements = reactive([
    {
        type: 'line',
        name: '线条一',
        color: '#fe39aa',
        dotList: [
            {
                x: 10,
                y: 10,
                radius: radius,
                color: 'red'
            },
            {
                x: 50,
                y: 50,
                radius: radius,
                color: 'blue'
            }
        ]
    },
    {
        type: 'rect',
        name: '矩形一',
        color: '#fe39aa',
        fillColor: '#e60012',
        dotList: [
            { x: 330, y: 140, radius: radius, color: 'red' },
            { x: 450, y: 140, radius: radius, color: 'blue' },
            { x: 450, y: 200, radius: radius, color: 'green' },
            { x: 330, y: 200, radius: radius, color: '#ffff00' },
        ]
    },
    {
        type: 'square',
        name: '正方形一',
        color: '#fe39aa',
        fillColor: '#2e59a7',
        dotList: [
            { x: 200, y: 300, radius: radius, color: 'red' },
            { x: 300, y: 300, radius: radius, color: 'blue' },
            { x: 300, y: 400, radius: radius, color: 'green' },
            { x: 200, y: 400, radius: radius, color: '#ffff00' },
        ]
    },
])
const onDown = (e) => {
    const mouseX = e.clientX - rect.left;
    const mouseY = e.clientY - rect.top;
    console.log('按下', mouseX, mouseY)
    // 从后向前检测（优先上层元素）
    for (let i = elements.length - 1; i >= 0; i--) {
        const el = elements[i];
        /**
         * 分为两种，一种线段，一种多边形
         * 线段判断是否在端点内，是的话控制端点，然后判断是否在线上，是的话控制整个线段
         * 多边形判断是否在端点内，是的话控制端点，然后判断是否在多边形内，是的话控制整个多边形
         * 
         */
        if (el.type === 'line') {
            const onCircle = setMoveDotInfo('line', el, i, mouseX, mouseY)
            if (onCircle) return
            // 选择线段整体
            if (isHitLine(mouseX, mouseY, el)) {
                console.log('onDown在线上')
                setMoveOverallInfo('line', el, i, mouseX, mouseY)
                return;
            }
        
        } else if (el.type === 'rect') {
            const onCircle = setMoveDotInfo('rect', el, i, mouseX, mouseY)
            if (onCircle) return
            // 选择矩形整体
            if (isHitPolygon(mouseX, mouseY, el)) {
                console.log('onDown在矩形内')
                setMoveOverallInfo('rect', el, i, mouseX, mouseY)
                return;
            }
        } else if (el.type === 'square') {
            const onCircle = setMoveDotInfo('square', el, i, mouseX, mouseY)
            if (onCircle) return
            // 选择正方形整体
            if (isHitPolygon(mouseX, mouseY, el)) {
                console.log('onDown在正方形内')
                setMoveOverallInfo('square', el, i, mouseX, mouseY)
                return;
            }
        }

    }

}

const setControlOptions = (el, draggingOverall, type, forCirculation, mouseX, mouseY) => {
    const { j, i, dotList } = forCirculation
    const elDot = dotList[j];

    dragTarget.value = elDot;
    isDraggingOverall.value = draggingOverall;
    isDragging.value = type

    offsetX.value = mouseX - dotList[j].x;
    offsetY.value = mouseY - dotList[j].y;
    if (type === 'rect') {
        dragRectDots = dotList.filter(dot => {
            if (dot.x !== elDot.x && dot.y !== elDot.y) {
                fixedPoint = dot
            }
            return (dot.x === elDot.x || dot.y === elDot.y) && !(dot.x === elDot.x && dot.y === elDot.y)
        })
    } else if (type === 'square') {
        // 当前点的上一个点和下一个点
        dragRectDots = [dotList[(j + 1) % 4], dotList[(j + 3) % 4]]
        fixedPoint = dotList[(j + 2) % 4]
    }
    const a = dotList.splice(j + 1);
    dotList.unshift(...a)
    elements.splice(i, 1);
    elements.push(el);
    render();
}

const setMoveOverallInfo = (type, el, i, mouseX, mouseY) => {
    dragTarget.value = el;
    offsetX.value = mouseX;
    offsetY.value = mouseY;
    isDraggingOverall.value = true;
    isDragging.value = type
    elements.splice(i, 1);
    elements.push(el);
    render();
}
const onUp = (e) => {
    isDragging.value = ''
    dragTarget.value = null;
    console.log('elements', elements)

}
const onLeave = (e) => {
    if (!isDragging.value) return
    if (!dragTarget.value) return;
    setNewPos(e);
    render();
    isDragging.value = ''
    dragTarget.value = null;
    console.log('left')
}
/** 移动 */
const onMove = (e) => {
    if (!isDragging.value) return
    if (!dragTarget.value) return;
     console.log('移动', e)
    setNewPos(e);
    render();
}
const setNewPos = (e) => {
    // console.log('设置新的位置', dragTarget.value)
    if (!isDraggingOverall.value) {

        // x的值要大于0，小于canvas的宽度，y的值要大于0，小于canvas的高度
        const x = e.clientX - rect.left - offsetX.value < 0 ? 0 : e.clientX - rect.left - offsetX.value > canvas.width ? canvas.width : e.clientX - rect.left - offsetX.value;
        const y = e.clientY - rect.top - offsetY.value < 0 ? 0 : e.clientY - rect.top - offsetY.value > canvas.height ? canvas.height : e.clientY - rect.top - offsetY.value;
        dragTarget.value.x = x;
        dragTarget.value.y = y;

        if (isDragging.value === 'rect') {
            dragRectDots.forEach(dot => {
                let dx = x, dy = y;

                if (dot.x === fixedPoint.x) {
                    dx = fixedPoint.x
                } else if (dot.y === fixedPoint.y) {
                    dy = fixedPoint.y
                }
                dot.x = dx
                dot.y = dy
            })
        } else if (isDragging.value === 'square') {
            // 拉伸正方形的一个点，该点相邻的那两个点也要移动
            updateAdjacentPoints(x, y)
        }
    } else {
        console.log(dragTarget.value, offsetY.value)
        console.log(e.clientY, rect.top)
        // 整体移动线段
        const dx = e.clientX - rect.left - offsetX.value;
        const dy = e.clientY - rect.top - offsetY.value;
        offsetX.value = e.clientX - rect.left;
        offsetY.value = e.clientY - rect.top;
        console.log('xxxxx', offsetY.value)
        dragTarget.value.dotList.forEach(el => {
            el.x += dx;
            el.y += dy;
        });

    }
}
/**
 * 碰撞检测（线段） 判断点击点是否在给定线段的一定范围内
 *
 * @param x 点击点的 x 坐标
 * @param y 点击点的 y 坐标
 * @param element 包含线段起点和终点的对象
 * @param range 点击点到线段的最大允许距离，默认为 12
 * @returns 点击点是否在给定范围内，返回布尔值
 */
const isHitLine = (x, y, element, range = 12) => {
    const [start, end] = element.dotList;

    // 计算线段的长度
    const dx = end.x - start.x;
    const dy = end.y - start.y;
    const lineLength = Math.sqrt(dx * dx + dy * dy);

    // 如果线段长度为0（即起点和终点相同），直接计算点到点的距离
    if (lineLength === 0) {
        return Math.sqrt((x - start.x) ** 2 + (y - start.y) ** 2) <= range;
    }

    // 计算点到线段的垂直距离的投影长度
    const t = ((x - start.x) * dx + (y - start.y) * dy) / (lineLength * lineLength);

    // 确保投影点在线段上
    if (t < 0 || t > 1) {
        return false;
    }

    // 计算投影点在线段上的坐标
    const px = start.x + t * dx;
    const py = start.y + t * dy;

    // 计算点击点到投影点的距离
    const distanceToProjection = Math.sqrt((x - px) ** 2 + (y - py) ** 2);

    return distanceToProjection <= range;
}
/** 碰撞检测（圆） */
const isHitCircle = (x, y, element) => {
    const dx = x - element.x;
    const dy = y - element.y;
    return (dx * dx + dy * dy) <= Math.pow(element.radius, 2);
}

/** 碰撞检测（多边形）（矩形） */
const isHitPolygon = (x, y, element) => {
    const points = element.dotList;
    let inside = false;
    for (let i = 0, j = points.length - 1; i < points.length; j = i++) {
        if (((points[i].y <= y && y < points[j].y) || (points[j].y <= y && y < points[i].y)) &&
            (x < (points[j].x - points[i].x) * (y - points[i].y) / (points[j].y - points[i].y) + points[i].x)) {
            inside = !inside;
        }
    }
    return inside;
}

/** 绘制多边形 */
const drawPolygon = (polygonInfo) => {
    ctx.beginPath()
    const LineColor = polygonInfo.color
    polygonInfo.dotList.forEach(({ x, y, color }, index) => {
        if (index === 0) {
            ctx.moveTo(x, y)
        } else {
            ctx.lineTo(x, y)
        }
    })
    ctx.strokeStyle = LineColor
    ctx.closePath();
    ctx.fillStyle = polygonInfo.fillColor || LineColor
    ctx.fill()
    ctx.strokeStyle = polygonInfo.color || LineColor
    ctx.stroke()

    // 计算多边形的中间位置，绘制元素标签
    const midX = polygonInfo.dotList.reduce((acc, { x }) => acc + x, 0) / polygonInfo.dotList.length
    const midY = polygonInfo.dotList.reduce((acc, { y }) => acc + y, 0) / polygonInfo.dotList.length
    drawText(polygonInfo.name, midX, midY)

    polygonInfo.dotList.forEach(({ x, y, radius, color }, index) => {
        drawCircle(x, y, radius, color)
    })
}
/** 绘制线 */
const drawLine = (lineInfo) => {
    const [start, end] = lineInfo.dotList
    const color = lineInfo.color

    ctx.beginPath()
    ctx.moveTo(start.x, start.y)
    ctx.lineTo(end.x, end.y)
    ctx.lineWidth = 2;
    ctx.strokeStyle = color
    ctx.stroke()
    // 计算两点的中间位置，绘制元素标签
    const midX = (start.x + end.x) / 2
    const midY = (start.y + end.y) / 2
    drawText(lineInfo.name, midX, midY)
    // 绘制两点连线上的圆点

    drawCircle(start.x, start.y, start.radius, (start.color || color))
    drawCircle(end.x, end.y, end.radius, (end.color || color))
}
/** 画圆点 */
const drawCircle = (x, y, r, color) => {
    ctx.beginPath(); //开始路径
    ctx.arc(x, y, r, 0, Math.PI * 2); //画圆
    ctx.fillStyle = color
    // ctx.lineWidth = 2;
    ctx.fill();
    // ctx.stroke(); //描边
}

/**
 * 在画布上绘制文本
 *
 * @param text 要绘制的文本
 * @param x 文本绘制的起始 x 坐标
 * @param y 文本绘制的起始 y 坐标
 * @param option 可选参数，用于自定义文本绘制样式
 *                   - color: 文本颜色，默认为 '#000'
 *                   - size: 文本字体大小，默认为 12
 *                   - center: 是否将文本居中对齐，默认为 true
 */
const drawText = (text, x, y, option) => {
    const defaultOption = { color: '#fff', size: 12, center: true }
    option = Object.assign(defaultOption, option)
    // 绘制元素标签
    ctx.fillStyle = option.color
    ctx.font = `${option.size}px Arial`
    if (option.center) {
        ctx.textAlign = 'center'
    }
    ctx.fillText(text, x, y)
}

// 绘制所有元素
const render = () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);


    elements.forEach(el => {
        switch (el.type) {
            case 'circle':
                drawCircle(el.x, el.y, el.radius, el.color)
                break;
            case 'line':
                drawLine(el)
                break;
            case 'polygon':
                drawPolygon(el)
                break;
            case 'rect':
                drawPolygon(el)
                break;
            case 'square':
                drawPolygon(el)
                break;
            default:
                console.error(`未知的元素类型：${el.type}`)
        }
    });
}
const initCanvas = () => {
    const canvas = canvasRef.value
    canvas.width = '800';
    canvas.height = '800';
    ctx = canvasRef.value.getContext('2d')  //context代表的是一支笔


    render()
}

onMounted(() => {
    canvas = canvasRef.value
    rect = canvas.getBoundingClientRect();

    initCanvas()
})
</script>

<style scoped>
.common-layout {
    background: gray;
}

.canvas_css {
    background: #110909;
    margin: 20px;
}
</style>
