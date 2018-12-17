---
title: Canvas API
date: 2018-09-18 19:58:59
tags:
- canvas
---

## 矩形

1. fillRect(x, y, width, height) 填充矩形
2. strokeRect(x, y, width, height) 绘制矩形边框
3. clearRect(x, y, width, height) 清除指定矩形区域，让清除部分完全透明。
4. rect(x, y, width, height) 绘制一个左上角坐标为（x,y），宽高为width以及height的矩形。

[绘制矩形 DEMO](https://jsfiddle.net/yacan8/sbLnkd9o/)

## 绘制路径

1. beginPath() 新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径。
2. closePath() 闭合路径，闭合路径之后图形命令又重新指向到上下文中。
3. stroke() 通过线条来绘制图形轮廓。
3. fill() 通过填充路径的内容区域生成实心图形。

当你调用fill()函数时，所有没有闭合的形状都会自动闭合，所以你不需要调用closePath()函数。但是调用stroke()时不会自动闭合。

[绘制三角形 DEMO](https://jsfiddle.net/yacan8/g09j2fc8/)

## 圆弧

1. arc(x, y, radius, startAngle, endAngle, anticlockwise) 画一个以（x,y）为圆心的以radius为半径的圆弧（圆），从startAngle开始到endAngle结束，按照anticlockwise给定的方向（默认为顺时针）来生成。
2. arcTo(x1, y1, x2, y2, radius) 根据给定的控制点和半径画一段圆弧，再以直线连接两个控制点。

arc()函数中表示角的单位是弧度，不是角度。角度与弧度的js表达式: 弧度=(Math.PI/180)*角度。

[圆弧DEMO](https://jsfiddle.net/yacan8/gamtc6v0/)

## 二次贝塞尔曲线及三次贝塞尔曲线

1. quadraticCurveTo(cp1x, cp1y, x, y) 绘制二次贝塞尔曲线，cp1x,cp1y为一个控制点，x,y为结束点。
2. bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y) 绘制三次贝塞尔曲线，cp1x,cp1y为控制点一，cp2x,cp2y为控制点二，x,y为结束点。

[二次贝塞尔曲线DEMO](https://jsfiddle.net/yacan8/wunr4q3L/)
[三次贝塞尔曲线DEMO](https://jsfiddle.net/yacan8/asbd92zy/)

## 色彩

1. fillStyle = color 设置图形的填充颜色。
2. strokeStyle = color 设置图形轮廓的颜色。

支持使用 rgba 形式。

[fillStyle DEMO](https://jsfiddle.net/yacan8/q043f9j8/1/)
[strokeStyle DEMO](https://jsfiddle.net/yacan8/demcxsky/1/)

## 透明度

1. globalAlpha = transparencyValue 这个属性影响到 canvas 里所有图形的透明度，有效的值范围是 0.0 （完全透明）到 1.0（完全不透明），默认是 1.0。

globalAlpha 属性在需要绘制大量拥有相同透明度的图形时候相当高效。相反，如果对单个元素添加透明度，推荐使用 rgba 形式。

[透明度 DEMO](https://jsfiddle.net/yacan8/bx63L72t/1/)
[rgba() DEMO](https://jsfiddle.net/yacan8/bx63L72t/2/)

## 移动触笔

1. moveTo(x, y) 将笔触移动到指定的坐标x以及y上。

当canvas初始化或者beginPath()调用后，你通常会使用moveTo()函数设置起点。我们也能够使用moveTo()绘制一些不连续的路径。

[移动触笔 DEMO](https://jsfiddle.net/yacan8/0mb4jc5a/)

## 线 

1. lineTo(x, y) 绘制一条从当前位置到指定x以及y位置的直线。

开始点也可以通过moveTo()函数改变。

[绘制两个三角形 DEMO](https://jsfiddle.net/yacan8/pqmx92nf/)

### 线型

1. lineWidth = value 设置线条宽度。
2. lineCap = type 设置线条末端样式。
选项：
butt 线段末端以方形结束。
round 线段末端以圆形结束。
square 线段末端以方形结束，但是增加了一个宽度和线段相同，高度是线段厚度一半的矩形区域。
3. lineJoin = type 设定线条与线条间接合处的样式。
选项：
round 通过填充一个额外的，圆心在相连部分末端的扇形，绘制拐角的形状。 圆角的半径是线段的宽度。
bevel 在相连部分的末端填充一个额外的以三角形为底的区域， 每个部分都有各自独立的矩形拐角。
miter 通过延伸相连部分的外边缘，使其相交于一点，形成一个额外的菱形区域。这个设置可以通过 miterLimit 属性看到效果。
4. miterLimit = value 限制当两条线相交时交接处最大长度；所谓交接处长度（斜接长度）是指线条交接处内角顶点到外角顶点的长度。
5. getLineDash() 返回一个包含当前虚线样式，长度为非负偶数的数组。
6. setLineDash(segments) 设置当前虚线样式。例：ctx.setLineDash([4, 16]);
7. lineDashOffset = value 设置虚线样式的起始偏移量。

[lineWidth DEMO](https://jsfiddle.net/yacan8/kx79j1mh/)
[lineCap DEMO](https://jsfiddle.net/yacan8/epbfmoqz/)
[lineJoin DEMO](https://jsfiddle.net/yacan8/awkqzhpb/)

## 渐变

1. createLinearGradient()方法创建一个沿参数坐标指定的直线的渐变。这个方法返回 CanvasGradient。
2. createRadialGradient() 是 Canvas 2D API 根据参数确定两个圆的坐标，绘制放射性渐变的方法。这个方法返回 CanvasGradient。

3. gradient.addColorStop(position, color) addColorStop 方法接受 2 个参数，position 参数必须是一个 0.0 与 1.0 之间的数值，表示渐变中颜色所在的相对位置。例如，0.5 表示颜色会出现在正中间。color 参数必须是一个有效的 CSS 颜色值（如 #FFF， rgba(0,0,0,1)，等等）。

[createLinearGradient DEMO](https://jsfiddle.net/yacan8/q9bftno1/1/)
[createRadialGradient DEMO](https://jsfiddle.net/yacan8/u4dow7Lq/)


## 图案样式

1. createPattern(image, type) Image 可以是一个 Image 对象的引用，或者另一个 canvas 对象。Type 必须是下面的字符串值之一：repeat，repeat-x，repeat-y 和 no-repeat。

如：
```js
var img = new Image();
img.src = 'someimage.png';
var ptrn = ctx.createPattern(img,'repeat');
```

与 drawImage 有点不同，你需要确认 image 对象已经装载完毕，否则图案可能效果不对的。

[createPattern DEMO](https://jsfiddle.net/yacan8/xb4c13za/)

## 阴影

1. shadowOffsetX = float 设定阴影在 X 轴的延伸距离。
2. shadowOffsetY = float 设定阴影在 Y 轴的延伸距离。
3. shadowBlur = float 用于设定阴影的模糊程度，其数值并不跟像素数量挂钩，也不受变换矩阵的影响，默认为 0。
4. shadowColor = color 是标准的 CSS 颜色值，用于设定阴影颜色效果，默认是全透明的黑色。

[文字阴影 DEMO](https://jsfiddle.net/yacan8/w23eovym/)

## 绘制文本 

1. fillText(text, x, y [, maxWidth]) 在指定的(x,y)位置填充指定的文本，绘制的最大宽度是可选的.
2. strokeText(text, x, y [, maxWidth]) 在指定的(x,y)位置绘制文本边框，绘制的最大宽度是可选的.
3. font = value 这个字符串使用和 CSS font 属性相同的语法. 默认的字体是 10px sans-serif。
4. textAlign = value 文本对齐选项. 可选的值包括：start, end, left, right or center. 默认值是 start。
5. textBaseline = value 基线对齐选项. 可选的值包括：top, hanging, middle, alphabetic, ideographic, bottom。默认值是 alphabetic。
6. direction = value 文本方向。可能的值包括：ltr, rtl, inherit。默认值是 inherit。
7. measureText() 将返回一个 TextMetrics对象的宽度、所在像素，这些体现文本特性的属性。

[textBaseline DEMO](https://jsfiddle.net/yacan8/Lyw1rga0/)

## 使用图片

canvas的API可以使用下面这些类型中的一种作为图片的源：

1. HTMLImageElement 这些图片是由Image()函数构造出来的，或者任何的\<img\>元素
2. HTMLVideoElement 用一个HTML的 \<video\>元素作为你的图片源，可以从视频中抓取当前帧作为一个图像
3. HTMLCanvasElement 可以使用另一个 \<canvas\> 元素作为你的图片源。
4. ImageBitmap 这是一个高性能的位图，可以低延迟地绘制，它可以从上述的所有源以及其它几种源中生成。

这些源统一由 [CanvasImageSource](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasImageSource) 类型来引用。

例：
```js
var img = new Image();   // 创建img元素
img.onload = function(){
  // 执行drawImage语句
}
img.src = 'myImage.png'; // 设置图片源地址
```

### 绘制图片

1. drawImage(image, x, y) 其中 image 是 image 或者 canvas 对象，x 和 y 是其在目标 canvas 里的起始坐标。
2. drawImage(image, x, y, width, height) 缩放 Scaling，这个方法多了2个参数：width 和 height，这两个参数用来控制 当向canvas画入时应该缩放的大小
3. drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight) 第一个参数和其它的是相同的，都是一个图像或者另一个 canvas 的引用。其它8个参数最好是参照右边的图解，前4个是定义图像源的切片位置和大小，后4个则是定义切片的目标显示位置和大小。

## 状态的保存和恢复

1. save()
2. restore()

save 和 restore 方法是用来保存和恢复 canvas 状态的，都没有参数。Canvas 的状态就是当前画面应用的所有样式和变形的一个快照。

Canvas状态存储在栈中，每当save()方法被调用后，当前的状态就被推送到栈中保存。一个绘画状态包括：

* 当前应用的变形（即移动，旋转和缩放，见下）
* strokeStyle, fillStyle, globalAlpha, lineWidth, lineCap, lineJoin, miterLimit, shadowOffsetX, shadowOffsetY, shadowBlur, shadowColor, globalCompositeOperation 的值
* 当前的裁切路径（clipping path）

每一次调用 restore 方法，上一个保存的状态就从栈中弹出，所有设定都恢复。

[save 和 restore DEMO](https://jsfiddle.net/yacan8/p7d34yte/)

## 变形

1. translate(x, y) 偏移 translate 方法接受两个参数。x 是左右偏移量，y 是上下偏移量。
2. rotate(angle) 这个方法只接受一个参数：旋转的角度(angle)，它是顺时针方向的，以弧度为单位的值。
3. scale(x, y) scale 方法接受两个参数。x,y 分别是横轴和纵轴的缩放因子，它们都必须是正值。值比 1.0 小表示缩小，比 1.0 大则表示放大，值为 1.0 时什么效果都没有。
4. transform(m11, m12, m21, m22, dx, dy) 
这个函数的参数，各自代表：
m11：水平方向的缩放，
m12：水平方向的倾斜偏移，
m21：竖直方向的倾斜偏移，
m22：竖直方向的缩放，
dx：水平方向的移动，
dy：竖直方向的移动。
5. setTransform(m11, m12, m21, m22, dx, dy) 这个方法会将当前的变形矩阵重置为单位矩阵，然后用相同的参数调用 transform 方法。
6. resetTransform() => setTransform(1, 0, 0, 1, 0, 0)。

[translate DEMO](https://jsfiddle.net/yacan8/vs82kham/)
[rotate DEMO](https://jsfiddle.net/yacan8/xgwtapbq/)


## 组合

1. globalCompositeOperation = type 这个属性设定了在画新图形时采用的遮盖策略，其值是一个标识12种遮盖方式的字符串，详情 type 参见 [Compositing 示例](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Compositing/Example);

## 裁切路径

裁切路径和普通的 canvas 图形差不多，不同的是它的作用是遮罩，用来隐藏不需要的部分。

如果和 globalCompositeOperation 属性作一比较，它可以实现与 source-in 和 source-atop差不多的效果。

1. clip() 方法来创建一个新的裁切路径。

[clip DEMO](https://jsfiddle.net/yacan8/8vabj1yp/)

## 动画

### 动画的基本步骤

你可以通过以下的步骤来画出一帧:

1. 清空 canvas。除非接下来要画的内容会完全充满 canvas （例如背景图），否则你需要清空所有。最简单的做法就是用 clearRect 方法。
保存 canvas 状态
如果你要改变一些会改变 canvas 状态的设置（样式，变形之类的），又要在每画一帧之时都是原始状态的话，你需要先保存一下。

2. 绘制动画图形（animated shapes）。这一步才是重绘动画帧。

3. 恢复 canvas 状态。如果已经保存了 canvas 的状态，可以先恢复它，然后重绘下一帧。

### 操控动画

1. setInterval(function, delay) 当设定好间隔时间后，function会定期执行。
2. setTimeout(function, delay) 在设定好的时间之后执行函数。
3. requestAnimationFrame(callback) 告诉浏览器你希望执行一个动画，并在重绘之前，请求浏览器执行一个特定的函数来更新动画。

[太阳系的动画 DEMO](https://jsfiddle.net/yacan8/0x1Lft57/1/)
[动画时钟 DEMO](https://jsfiddle.net/yacan8/6834fyrk/)
[循环全局 DEMO](https://jsfiddle.net/yacan8/21Lwqvg8/)

#### 小球 DEMO

[首个预览](https://jsfiddle.net/yacan8/0n72q6pg/)
[加速度](https://jsfiddle.net/yacan8/hL9y6tkc/)
[长尾效果](https://jsfiddle.net/yacan8/p8qLed60/)
[添加鼠标控制](https://jsfiddle.net/yacan8/dqat0w9h/)

## 像素操作

### ImageData对象

ImageData 对象中存储着canvas对象真实的像素数据，它包含以下几个只读属性：

1. width 图片宽度，单位是像素
2. height 图片高度，单位是像素
3. data [Uint8ClampedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray) 类型的一维数组，包含着RGBA格式的整型数据，范围在0至255之间（包括255）。

data属性返回一个 Uint8ClampedArray，它可以被使用作为查看初始像素数据。每个像素用4个1bytes值(按照红，绿，蓝和透明值的顺序; 这就是"RGBA"格式) 来代表。每个颜色值部份用0至255来代表。每个部份被分配到一个在数组内连续的索引，左上角像素的红色部份在数组的索引0位置。像素从左到右被处理，然后往下，遍历整个数组。

### 创建ImageData对象

去创建一个新的，空白的ImageData对象，你应该会使用createImageData() 方法。有2个版本的createImageData()方法。

```js
var myImageData = ctx.createImageData(width, height);
```

得到场景像素数据，为了获得一个包含画布场景像素数据的ImageData对像，你可以用getImageData()方法：

```js
var myImageData = ctx.getImageData(left, top, width, height);
```

这个方法会返回一个ImageData对象，它代表了画布区域的对象数据，此画布的四个角落分别表示为(left, top), (left + width, top), (left, top + height), 以及(left + width, top + height)四个点。这些坐标点被设定为画布坐标空间元素。

在场景中写入像素数据，可以用putImageData()方法去对场景进行像素数据的写入。

```js
ctx.putImageData(myImageData, dx, dy);
```

dx和dy参数表示你希望在场景内左上角绘制的像素数据所得到的设备坐标。

### 保存图片

1. canvas.toDataURL(type, quality) 创建一个base64的type类型图片，你可以有选择地提供从0到1的 quality 品质量，1表示最好品质，0基本不被辨析但有比较小的文件大小。

2. canvas.toBlob(callback, type, quality) 这个创建了一个在画布中的代表图片的Blob对像，callback：回调函数，可获得一个单独的Blob对象参数。type 可选 DOMString类型，指定图片格式，默认格式为image/png，quality：图片质量。


[颜色选择器 DEMO](https://jsfiddle.net/yacan8/zeynLp6v/2/)
[图片灰度和反相颜色](https://jsfiddle.net/yacan8/qaz5t83v/1/)

