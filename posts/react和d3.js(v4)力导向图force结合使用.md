---
title: react和d3.js(v4)力导向图force结合使用
date: 2017-11-29 15:21:24
tags:
- react
- d3.js
- 力导向图
---

前段时间由于性能要求，需把项目d3的版本从v3升级到v4，据了解d3由于在v4版本之前是没有进行模块化的，所以v3代码的扩展性是比较差的，考虑到长远之计，d3在v4版本算是对代码进行了模块化的重构吧，给开发者提供了一些可定制化的东西，所有api变化较大，这个坑还需各种研究文档才能填完，好了，下面开始我的表演了。
### 初始化force布局
初始化函数从v3的`d3.layout.force()`变成v4的`d3.forceSimulation()`，部分参数设置方式如下:
```js
this.force = d3.forceSimulation().alphaDecay(0.1) // 设置alpha衰减系数
                .force("link", d3.forceLink().distance(100)) // distance为连线的距离设置
                .force('collide', d3.forceCollide().radius(() => 30)) // collide 为节点指定一个radius区域来防止节点重叠。
                .force("charge", d3.forceManyBody().strength(-400))  // 节点间的作用力
```
为布局添加点和线
```js
this.force.nodes(nodes)   // 节点数据
          .force('link', d3.forceLink(links).distance(150));  // 连线数据 distance为连线的距离设置
          .alpha(1);  // 设置alpha值，让里导向图有初始动力
          .restart();   // 启动仿真计时器
```
由于在v4版本中nodes的`x`、`y`坐标和加速度`vx`、`vy`只在nodes中计算一次，所有在变成有节点或连线增加的时候，必须重新执行一次`force.nodes(nodes)`和`force('link', d3.forceLink(links))`，初始化节点的数据结构。如果在v3版本中，只需在布局初始化时执行即可，在d3会在每次`force.start()`方法执行时重新初始化一次节点和连线的数据结构，这是一个特别需要注意的地方，另外在v4版本中`start`方法被遗弃，需使用`restart`方法。

### react部分
将节点的dom结构交给react来控制，方便在节点上添加事件。以下为svg渲染部分代码。
```js
render() {
    const { width, height, nodes, links, scale, translate, selecting, grabbing } = this.props.store;
    return (
      <svg id="svg" ref="svg" width={width} height={height}
        className={cn({
          grab: !selecting && !grabbing,
          grabbing: !selecting && grabbing
        })}
        >
        <g id="outg" ref="outg" transform={`translate(${translate})scale(${scale})`}>
          <g ref="lines" className="lines">
            links.map(link => (
                <line
                  key={`${link.source.uid}_${link.target.uid}`}
                  ref={child => this.links[`${link.source.uid}_${link.target.uid}`] = child}
                  x1={link.source.x}
                  y1={link.source.y}
                  x2={link.target.x}
                  y2={link.target.y}/>
              ))
          </g>
          <g ref="nodes" className="nodes">
            {
              nodes.map(node => (
                <Node key={node.uid}
                  node={node}
                  store={this.props.store}
                  addRef={child => this.nodes[node.uid] = child}/>
              ))
            }
          </g>
        </g>
      </svg>
    );
  }
```
Node.js 节点
以下为Node Component部分代码

```js
class Node extends Component {
  render() {
    const { node, addRef, store } = this.props;
    const { force } = store;
    return (
      <g className="node"
        ref={child => {
          this._node = child;
          addRef(child);
        }}
        transform={`translate(${node.x || width / 2},${node.y || height / 2})`}
        >
        <g id={node.nodeIndex}>
          // 节点图片dom
        </g>
        {
          node.locked && (
            <Lock
              x={10}
              y={10}
              release={() => {   // 解锁节点
                node.fixed = false;
                node.locked = false;
                node.fx = null;   // 当节点的fx、fy都为null时，节点处于活动状态
                node.fy = null;   
                force.alpha(0.3).restart();  // 释放锁定节点时需设置alpha值并重启计时器，使得布局可以运动。
              }}
              />
          )
        }
      </g>
    );
  }

  componentDidMount() {
    this._node.__data__ = this.props.node;  // 将node节点在d3内部存一份引用，让每次计时器更新的时候自动更改nodes列表中的数据
    d3.select(this._node)  // 各种事件
      .on('click', d => {
          // code
      })
  }
}
```

Lock.js 节点解除固定按钮。
```js
class Lock extends Component {
  render() {
    const { x, y, fixed } = this.props;
    return (
      <use
        ref="lock"
        xlinkHref="#lock"
        x={x}
        y={y}
        />
    );
  }

  componentDidMount() {
    const { release } = this.props;
    d3.select(this.refs.lock)
      .on('click', () => {
        d3.event.stopPropagation();
        release();
      });
  }
}
```
### 仿真计时器 `tick`
计时器函数，在仿真启动的过程中，计时器的每一帧都会改变一次之前我们在内部存的引用(`this._node.__data__ = this.props.node`)的node的数据的`x`值和`y`值，这时我们需要更新dom结构中的节点和线偏移量。
```js
force.on('tick', () => {
  nodes.forEach(node => {
    if (!node.lock) {
      d3.select(self.nodes[node.uid]).attr('transform', () => `translate(${node.x},${node.y})`);
    }
  });
  links.forEach(link => {
    d3.select(self.links[`${link.source.uid}_${link.target.uid}`])
      .attr('x1', () => link.source.x)
      .attr('y1', () => link.source.y)
      .attr('x2', () => link.target.x)
      .attr('y2', () => link.target.y);
  });
});
```
在计时器的每一帧中，仿真的`alpha`系数会不断削减，可通过`force.alpha()`来获取和设置`alpha`系数，削减速度由`alphaDecay`来决定，默认值为0.0228…，衰减系数可通过`force.alphaDecay()`来获取和设置，当`alpha`到达一个系数时，仿真将会停止，也就是`alpha`的目标系数`alphaTarget`，该值区间为[0,1]. 默认为0，可通过`force.alphaTarget()`来获取和设置，另外还有一个速度衰减系统`velocityDecay `，相当于摩擦力。区间为[0,1], 默认为0.4。在每次`tick`之后，节点的速度都会等于当前速度乘以`1-velocityDecay`，和`alpha`衰减类似，速度衰减越慢最终的效果越好，但是如果速度衰减过慢，可能会导致震荡。以上为`tick`过程的发生。需要注意的是，在v4版本中，`tick`事件的callback中不带任何参数，在v3版本的'tick'事件中，我们可通过`callback(e)`中的`e.alpha`来获取`alpha`值，而在v4版本中，`alpha`值只能通过`force.alpha()`来获取。
### 拖拽 Drag
创建拖拽操作
```js
let startTime = 0;
this.drag = d3.drag()
      .on('start', (d) => {
        startTime = (new Date()).getTime();
        d3.event.sourceEvent.stopPropagation();
        if (!d3.event.active) {
           this.force.alphaTarget(0.3).restart();  // 当前alpha值为0，需设置alphaTarget让节点动起来
        }
        d.fx = d.x;
        d.fy = d.y;
      })
      .on('drag', d => {
        this.grabbing = true;
        d.fx = d3.event.x;
        d.fy = d3.event.y;
      })
      .on('end', d => {
        const nowTime = (new Date()).getTime();
        if (!d3.event.active) {
           this.force.alphaTarget(0);  // 让alpha目标值值恢复为默认值0
        }
        if (nowTime - startTime >= 150) {  // 操作150毫秒的拖拽固定节点
          d.fixed = true;
          d.locked = true;
        }
        this.grabbing = false;
      });
```
将拖拽操作应用到指定的选择集。
```js
    d3.select('#outg').selectAll('.node').call(this.drag);
```
在内部，拖拽操作通过`selection.on`来为元素添加监听事件. 事件监听器使用 .drag 来标识这是一个拖拽事件。拖拽drag的v4版本与v3不同的是，v3通过`force.drag()`创建拖拽操作，拖拽过程事件使用`dragstart`、`drag`、`dragend`，在拖拽过程中d3内部自动设置`alpha`相关系数让节点运动起来，而在v4中版本中需要手动设置。

### 缩放 Zoom
在v4版本中，缩放操作通过`transform`对象进行，可以通过`d3.zoomTransform(selection.node())`获取指定节点的缩放状态，也可以通过`d3.event.transform`来获取当前正在缩放的节点的缩放状态。
与拖拽类似，需要先创建缩放操作。
```js
const self = this;
const outg = d3.select('#outg');
this.zoomObj = d3.zoom()
      .scaleExtent([0.2, 4]) // 缩放范围
      .on('zoom',() => {
        const transform = d3.event.transform;
        self.scale = transform.k;  // 保存当前缩放大小
        self.translate = [transform.x, transform.y];  // 保存当前便宜量
        outg.attr('transform', transform);   // 设置缩放和偏移量 transform对象自带toString()方法
      })
      .on('end', () => {
        // code
      })
```
将缩放操作应用于选择集，并取消双击操作
```js
const svg = d3.select('#svg');
svg.call(this.zoomObj).on('dblclick.zoom', null);
```
如果要禁止滚轮滚动缩放，可以在讲zoom事件应用于选择集之后移除zoom事件中的滚轮事件：
```js
svg.call(this.zoomObj).on("wheel.zoom", null);
```
当缩放事件被调用，`d3.event`会被设置为当前的zoom事件，`zoom event`对象由以下几部分组成:
* `target` - 当前的缩放zoom behavior。
* `type` - 事件类型:“start”, “zoom” 或者 “end”，参考 zoom.on。
* `transform` - 当前的zoom transform(缩放变换)。
* `sourceEvent` - 原始事件, 比如 mousemove 或 touchmove。
通过按钮缩放、定位视图。
```js
this.zoomObj.transform(d3.select('#svg'), d3.zoomIdentity.translate(newX,newY).scale(newScale))
```
在v3版本中，可以通过`zoom.scale(s)`和`zoom.translate(x, y)`设置缩放和偏移量后通过使用'zoom.event(selection)'方法应用到指定选择节点，而在v4中版本需要通过`d3.zoomIdentity`创建新`transform`对象，并通过`translate(x, y)`和`scale(s)`方法设置偏移量和缩放级别，然后将该`transform`应用到选择集中。另外也可以通过`zoom.translateBy(selection, x, y)`、`zoom.translateTo(selection, x, y)`、`zoom.scaleBy(selection, k)`、`zoom.scaleTo(selection, k)`方法进行变换。
### 小结
由于api变动较大，v3升级v4需要耐心看api，查看各个部分的变化，所以，升级需谨慎。最后附上[d3.js v4.0中文api](https://github.com/xswei/d3js_doc)。
