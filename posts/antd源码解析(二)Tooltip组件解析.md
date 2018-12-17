---
title: antd源码解读（二）Tooltip组件解析
date: 2018-05-01 20:00:13
tags:
- antd
- 前端
- react
---

### 引言
antd的Tooltip组件在[react-componment/trigger](https://github.com/react-component/trigger)的基础上进行封装，而组件Popover和Popconfirm是使用Tooltip组件的进行pop，在[react-componment](https://github.com/react-component)中，使用到组件`tc-trigger`的还有menu、select、dropdown、time-picker、calendar等，本文主要对`tc-trigger`源码进行解读。

### 结构
项目结构如下：

![项目结构](/images/antd源码解读（二）Tooltip组件解析/1.jpeg)

* index.js，负责外层封装，负责事件绑定与dom渲染控制。
* LazyRenderBox.js，pop内容懒加载warp。
* mock.js 未使用。
* Popup.js，pop的warp，负责控制pop的对齐、动画、宽高。
* PopupInner.js，pop内容warp。

### index.js
从render方法入手，需要渲染控制pop显示的节点和pop内容节点两个节点，而pop内容节点一般渲染到body里面，不属于控制pop显示的节点内，render方法代码如下：
```html
  const trigger = React.cloneElement(child, newChildProps);
  if (!IS_REACT_16) {
    return (
      <ContainerRender
        parent={this}
        visible={popupVisible}
        autoMount={false}
        forceRender={props.forceRender}
        getComponent={this.getComponent}
        getContainer={this.getContainer}
      >
        {({ renderComponent }) => {
          this.renderComponent = renderComponent;
          return trigger;
        }}
      </ContainerRender>
    );
  }

  let portal;
  // prevent unmounting after it's rendered
  if (popupVisible || this._component || props.forceRender) {
    portal = (
      <Portal
        key="portal"
        getContainer={this.getContainer}
        didUpdate={this.handlePortalUpdate}
      >
        {this.getComponent()}
      </Portal>
    );
  }

  return [
    trigger,
    portal,
  ];
```
可以看到，index.js渲染了两个节点，trigger和portal，trigger即为通过事件控制portal显示状态的节点，如果react的版本不是16以上，返回ContainerRender组件，ContainerRender组件来自`rc-util`，该组件主要做的事情就是使用`ReactDOM.unstable_renderSubtreeIntoContainer`函数，将pop内容渲染到trigger节点之外，与react16提供的API`createPortal`作用一致，如果是React16，返回了Portal组件，该组件正是利用了createPortal，将组件渲染到特定的dom节点内，但是不管是不是react16，都进行了pop渲染的判断，即`popupVisible || this._component || props.forceRender`，如果portal不显示且不强制第一次渲染`forceRender`，portal将不会被渲染到dom中，直到判断为真。

trigger节点通过props决定事件绑定情况，即通过`props.trigger`属性绑定事件情况，事件控制Popup组件的visible属性，这里就不详细说了。

### Popup.js
该组件是pop的warp，渲染在trigger节点之外，通过`ReactDOM.unstable_renderSubtreeIntoContainer`或`createPortal`指定渲染的目标节点，也是render方法入手：
```html
render() {
  return (
    <div>
      {this.getMaskElement()}
      {this.getPopupElement()}
    </div>
  );
}
```
返回两个内容，`getMaskElement`获取遮罩，`getPopupElement`返回Pop节点，getMaskElement这里就不说了，渲染的视觉效果，绑定了控制pop节点的事件。

getPopupElement返回pop节点，render返回代码如下:
```html
  <Animate
    component=""
    exclusive
    transitionAppear
    transitionName={this.getTransitionName()}
    showProp="xVisible"
  >
    <Align
      target={this.getTarget}
      key="popup"
      ref={this.saveAlignRef}
      monitorWindowResize
      xVisible={visible}
      childrenProps={{ visible: 'xVisible' }}
      disabled={!visible}
      align={align}
      onAlign={this.onAlign}
    >
      <PopupInner
        hiddenClassName={hiddenClassName}
        {...popupInnerProps}
      >
        {children}
      </PopupInner>
    </Align>
  </Animate>
```
Animate来自组件[rc-animate](https://github.com/react-component/animate)，主要负责显示状态切换时候的动态效果，其中原理是监听控制状态变化的prop属性，即代码中的`showProp="xVisible"`，当状态变化的时候，延时改变dom的class，一般会有三个状态，分别表示进入中enter-active，消失中leave-active，隐藏hidden三个状态，进入中状态会添加`transitionName-enter transitionName-enter-active`两个class，消失中会添加`transitionName-leave transitionName-leave-active`两个class，隐藏状态不添加class，transitionName通过外部传入。

Align来自组件[rc-align](https://github.com/react-component/align)，主要控制节点的相对于trigger的显示位置，根据传入的target与align决定最后PopupInner显示的位置，此处target是来自于index.js的trigger节点，align也是来自于index.js，主要由index.js的prop.popupPlacement、prop.popupAlign两个属性决定，即方向与偏移量。

最后是PopupInner组件，该组件是也就pop内容组件，内容通过LazyRenderBox包裹。。。

另外，Popup.js还有两个state，targetWidth与targetHeight，即pop的宽高，该属性如果设置有prop.stretch，则计算trigger真是dom节点的宽高，然后对齐。

### PopupInner.js
为隐藏状态下的pop添加hidden的class，并包裹懒加载组件LazyRenderBox。

### LazyRenderBox.js
只做一件事情，就是将popupInner的chidren进行包裹，当子节点数大于1时，包一层div以方便隐藏状态时候class控制，不用每个节点都添加hidden的class，关键如下：
```js
render() {
  const { hiddenClassName, visible, ...props } = this.props;
  if (hiddenClassName || React.Children.count(props.children) > 1) {
    if (!visible && hiddenClassName) {
      props.className += ` ${hiddenClassName}`;
    }
    return <div {...props}/>;
  }
  return React.Children.only(props.children);
}
```

### 最后
该组件主要的实现难点在于[rc-animate](https://github.com/react-component/animate)与[rc-align](https://github.com/react-component/align)，其他的主要在做事件绑定与class处理。
