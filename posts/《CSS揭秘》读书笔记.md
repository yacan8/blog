---
title: 《CSS揭秘》读书笔记 
date: 2018-03-20 16:41:44
tags:
- CSS
- 前端
---

## 第一章
1. 尽量减少代码重复：代码可维护性的最大要素是尽量减少改动时要编辑的地方。
2. 关于响应式设计：每个媒体查询都会增加成本。
3. 合理使用简写：合理使用简写是一种良好的防卫性编码方式，可以抵御未来的风险。如果只为某个属性提供一个值，那 它就会扩散并应用到列表中的每一项。
4. 关于CSS预处理：如果使用得当，它们在大型项目中可以让代码更加灵活。缺点，CSS 的文件体积和复杂度可能会失控，调试难度会增加，预处理器在开发过程中引入了一定程度的延时。

## 第二章 背景与边框
1. 半透明边框：hsla、rgba、background-clip处理。
2. 多重边框：box-shadow逗号分隔多重投影、outline+border双重边框。
3. 背景定位：background-position、background-origin背景定位模式、calc计算背景位置。
4. 边框与圆角：border-radius
5. 条纹背景：线性渐变linear-gradient及background-size背景大小
6. 复杂的背景图案：background-image与linear-gradient的逗号分隔、background-position结合使用。
7. 连续的图案边框：background、linear-gradient、background-size、background-clip、background-origin结合使用。

## 第三章 形状
1. 自适应椭圆： border-radius水平方向与垂直方向，斜杠。
2. 平行四边形：transform的skewX，文字反向skewX。
3. 菱形：transform的rotate。
4. 切片效果：直角切片linear-gradient、弧形切片radial-gradient，clip-path的polygon。
5. 梯形标签页：transform-origin、transform的perspective、rotateX使用。

## 第四章 视觉效果
1. 单侧投影：box-shadow。
2. 不规则投影：filter的drop-shadow。
3. 染色效果：filter的sepia、saturate、hue-rotate。background-blend-mode的luminosity。
4. 毛玻璃效果：filter的blur以及伪元素的使用。

## 第七章 布局与结构
1. 自适应内部元素：width的min-content使用。
2. 精确控制表格列宽：table-layout的fixed、text-overflow的ellipsis。
3. 根据兄弟元素的数量来设置样式：:nth-child(an+b)、:nth-last-child(an+b)、~选择器
4. 满幅的背景，定宽的内容：calc使用。
5. 垂直居中：定位方案、calc使用、transform。

## 第八章 过渡与动画
1. 缓慢动画：动画的基本使用、ease使用、transition-timing-function的cubic-bezier设置贝塞尔调速函数。
2. 逐帧动画：animation的steps设置逐帧调速函数使用。
3. 闪烁效果：animation-direction反正动画循环alternate、reverse、alternate-reverse。