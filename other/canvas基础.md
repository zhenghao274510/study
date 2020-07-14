# Canvas

Canvas 是 HTML5 提供的一个用于展示绘图效果的标签. Canvas 原意为画布, 在 HTML 页面中用于展示绘图效果. 最早 canvas 是苹果提出的一个方案, 今天已经在大多数浏览器中实现。

canvas 的使用领域

- 游戏
- 大数据可视化数据
- banner 广告
- 多媒体
- 模拟仿真
- 远程操作
- 图形编辑

判断浏览器是否支持 canvas 标签

```js
var canvas = document.getElementById('canvas')
if (canvas.getContext) {
  console.log('你的浏览器支持Canvas!')
} else {
  console.log('你的浏览器不支持Canvas!')
}
```

### canvas 的基本用法

1、使用 canvas 标签, 即可在页面中开辟一格区域，可以设置其宽高，宽高为 300 和 150

```html
<canvas></canvas>
```

2、获取 dom 元素 canvas

canvas 本身不能绘图. 是使用 JavaScript 来完成绘图. canvas 对象提供了各种绘图用的 api。

```js
var cas = document.querySelector('canvas')
```

3、通过 cas 获取上下文对象(画布对象!)

```js
var ctx = cas.getContext('2d')
```

4、通过 ctx 开始画画（设置起点 设置终点 连线-描边 ）

```js
ctx.moveTo(10, 10)
ctx.lineTo(100, 100)
ctx.stroke()
```

### 绘制线条

- 设置开始位置: context.moveTo( x, y ).
- 设置终点位置: context.lineTo( x, y ).
- 描边绘制: context.stroke().
- 填充绘制: context.fill().
- 闭合路径: context.closePath().

canvas 还可以设置线条的相关属性，如下：

- CanvasRenderingContext2D.lineWidth 设置线宽.
- CanvasRenderingContext2D.strokeStyle 设置线条颜色.
- CanvasRenderingContext2D.lineCap 设置线末端类型，'butt'( 默认 ), 'round', 'square'.
- CanvasRenderingContext2D.lineJoin 设置相交线的拐点， 'miter'(默认)，'round', 'bevel',
- CanvasRenderingContext2D.getLineDash() 获得线段样式数组.
- CanvasRenderingContext2D.setLineDash() 设置线段样式.
- CanvasRenderingContext2D.lineDashOffset 绘制线段偏移量.

封装一个画矩形的方法

```js
function myRect(ctxTmp, x, y, w, h) {
  ctxTmp.moveTo(x, y)
  ctxTmp.lineTo(x + w, y)
  ctxTmp.lineTo(x + w, y + h)
  ctxTmp.lineTo(x, y + h)
  ctxTmp.lineTo(x, y)
  ctxTmp.stroke()
}

var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')
myRect(ctx, 50, 50, 200, 200)
```

### 绘制矩形

- fillRect( x , y , width , height) 填充以(x,y)为起点宽高分别为 width、height 的矩形 默认为黑色
- stokeRect( x , y , width , height) 绘制一个空心以(x,y)为起点宽高分别为 width、height 的矩形
- clearRect( x, y , width , height ) 清除以(x,y)为起点宽高分别为 width、height 的矩形 为透明

### 绘制圆弧

绘制圆弧的方法有

- CanvasRenderingContext2D.arc()
- CanvasRenderingContext2D.arcTo()

6 个参数: x，y(圆心的坐标)，半径，起始的弧度(不是角度 deg)，结束的弧度，(bool 设置方向 ! )

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')

ctx.arc(100, 100, 100, 0, degToArc(360))
ctx.stroke()

// 角度转弧度
function degToArc(num) {
  return (Math.PI / 180) * num
}
```

### 绘制扇形

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')

ctx.arc(300, 300, 200, degToArc(125), degToArc(300))

// 自动连回原点
ctx.closePath()
ctx.stroke()

function degToArc(num) {
  return (Math.PI / 180) * num
}
```

### 制作画笔

1. 声明一个变量作为标识
2. 鼠标按下的时候，记录起点位置
3. 鼠标移动的时候，开始描绘并连线
4. 鼠标抬起的时候，关闭开关

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')

var isDraw = false
// 鼠标按下事件
cas.addEventListener('mousedown', function () {
  isDraw = true
  ctx.beginPath()
})

// 鼠标移动事件
cas.addEventListener('mousemove', function (e) {
  if (!isDraw) {
    // 没有按下
    return
  }
  // 获取相对于容器内的坐标
  var x = e.offsetX
  var y = e.offsetY
  ctx.lineTo(x, y)
  ctx.stroke()
})

cas.addEventListener('mouseup', function () {
  // 关闭开关了!
  isDraw = false
})
```

### 手动涂擦

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')

ctx.fillRect(0, 0, 600, 600)

// 开关
var isClear = false

cas.addEventListener('mousedown', function () {
  isClear = true
})

cas.addEventListener('mousemove', function (e) {
  if (!isClear) {
    return
  }
  var x = e.offsetX
  var y = e.offsetY
  var w = 20
  var h = 20
  ctx.clearRect(x, y, w, h)
})

cas.addEventListener('mouseup', function () {
  isClear = false
})
```

### 刮刮乐

1. 首先需要设置奖品和画布，将画布置于图片上方盖住，
2. 随机设置生成奖品。
3. 当手触摸移动的时候，可以擦除部分画布，露出奖品区。
html

```html
<div>
  <img src="./images/2.jpg" alt="" />
  <canvas width="600" height="600"></canvas>
</div>
```

css

```css
img {
  width: 600px;
  height: 600px;
  position: absolute;
  top: 10%;
  left: 30%;
}

canvas {
  width: 600px;
  height: 600px;
  position: absolute;
  top: 10%;
  left: 30%;
  border: 1px solid #000;
}
```

js

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')
var img = document.querySelector('img')
// 加一个遮罩层
ctx.fillStyle = '#ccc'
ctx.fillRect(0, 0, cas.width, cas.height)
setImgUrl()
// 开关
var isClear = false
cas.addEventListener('mousedown', function () {
  isClear = true
})
cas.addEventListener('mousemove', function (e) {
  if (!isClear) {
    return
  }
  var x = e.offsetX
  var y = e.offsetY
  ctx.clearRect(x, y, 30, 30)
})
cas.addEventListener('mouseup', function () {
  isClear = false
})

function setImgUrl() {
  var arr = ['./images/1.jpg', './images/2.jpg', './images/3.jpg', './images/4.jpg']
  // 0-3
  var random = Math.round(Math.random() * 3)
  img.src = arr[random]
}
```

### 在线画图

drawImage() 方法在画布上绘制图像、画布或视频。drawImage() 方法也能够绘制图像的某些部分，以及/或者增加或减少图像的尺寸。参数如下

* img	规定要使用的图像、画布或视频。
* sx	可选。开始剪切的 x 坐标位置。
* sy	可选。开始剪切的 y 坐标位置。
* swidth	可选。被剪切图像的宽度。
* sheight	可选。被剪切图像的高度。
* x	在画布上放置图像的 x 坐标位置。
* y	在画布上放置图像的 y 坐标位置。
* width	可选。要使用的图像的宽度。（伸展或缩小图像）
* height	可选。要使用的图像的高度。（伸展或缩小图像）

```js
var cas = document.querySelector('canvas')
var ctx = cas.getContext('2d')
// 先创建图片对象
var img = new Image()
img.src = './images/1.jpg'

// 图片加载完之后
img.onload = function () {
  ctx.drawImage(img, 206, 111, 32, 38, 100, 100, 32, 38)
}
```
