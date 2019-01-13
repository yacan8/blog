---
title: 从react专职到vue开发的项目准备
date: 2019年01月13日15:47:41
tags:
- react
- 前端
- vue
---

## 前言

首先，为什么我需要做这个项目准备工作呢？因为常年习惯React开发的我，最近突然接手了一个Vue项目，而之前基本没有过Vue的实践，这么突兀让还在沉溺于React开发的我进行Vue开发，甚是不习惯，那自然我需要想办法让Vue开发尽量与React相似，这样大概让自己在开发过程中更得心应手吧。

## 组件开发

### 特性对比

众所周知，Vue和React都有那么一个特性，那就是可以让我们进行组件化开发，这样可以让代码得到更好的重用以及解耦，在架构定位中这个应该叫纵向分层吧。但是，两个框架开发组件的写法都有所不同（这个不同是基于我的开发习惯），下面先看一下不同的地方。

首先是React，个人习惯于es6的写法（从来没用过es5的createClass的写法）：

```js
import React, { Component } from 'react';
import propTypes from 'prop-types';

export default class Demo extends Component {

  state = {
    text: 'hello world'
  };

  static propTypes = {
    title: PropTypes.String
  }

  static defaultProps = {
    title: 'React Demo'
  }

  setText = e => {
    this.setState({
      text: '点击了按钮'
    })
  }

  componentWillReveiveProps(nextProps) {
    console.log(`标题从 ${this.props.title} 变为了 ${nextProps.title}`)
  }

  render() {
    const { title } = this.props;
    const { text } = this.state;
    return <div>
      <h1>{title}</h1>
      <span>{text}<span>
      <button onClick={this.setText}>按钮<button>
    </div>
  }
}

```

下面是常见vue的写法：

```html
<template>
  <div>
    <h1>{{title}}</h1>
    <span>{{text}}<span>
    <button @click="setText">按钮</button>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
      default: 'Vue Demo'
    }
  },
  watch: {
    title(newTitle, oldTitle) {
      console.log(`标题从 ${oldTile} 变为了 ${newTitle}`)
    }
  },
  data() {
    return {
      text: 'hello world'
    }
  },
  methods: {
    setText(e) {
      this.text = '点击了按钮';
    }
  }
}
</script>
```

这里的视图渲染我们先忽略，下一节在详细对比。

prop对比：

* Vue的prop必须在props字段里声明。React的prop不强制声明，声明时也可以使用`prop-types`对其声明约束。
* Vue的prop声明过后挂在在组件的this下，需要的时候在this中获取。React的prop存在组件的props字段中，使用的时候直接在this.props中获取。

组件状态对比，Vue为data，React为state：

* Vue的状态data需要在组件的data字段中以函数的方式声明并返回一个对象。React的状态state可以直接挂载在组件的state字段下，在使用之前初始化即可。
* Vue的状态data声明后挂在在this下面，需要的是时候在this中获取。React的状态state存在组件的state字段中，使用的时候直接在this.state中获取。
* Vue的状态更新可以直接对其进行赋值，视图可以直接得到同步。React的状态更新必须使用`setState`，否则视图不会更新。

然后是组件方法对比：

* Vue的方法需要在`methods`字段下声明。React的方法用方法的方式声明在组件下即可。
* Vue与React使用方法的方式相同，因为都是挂载在组件中，直接在this中获取即可。

计算属性computed对比：

* Vue有计算属性在`computed`字段中声明。React中无计算属性特性，需要其他库如mobx辅助完成。
* Vue的计算属性声明后挂载在this下，需要的时候在this中获取。

监听数据对比：

* Vue中可以在`watch`字段中对prop、data、computed进行对比，然后做相应的操作。在React所有变化需要在声明周期`componentWillReveiveProps`中手动将state和prop进行对比。

对比完后发现，其实Vue给我的个人感觉就是自己在写配置，只不过配置是以函数的形式在写，然后Vue帮你把这些配置好的东西挂载到组件下面。而且prop、data、computed、方法所有都是挂载组件下，其实单单从js语法上很难以理解，比如说我在computed中，想获取data的text数据，使用的是this.text来获取，如果抛开vue，单单用js语法来看，其实this大多情况是指向computed对象的，所以个人觉得这样的语法是反面向对象的。

这个时候在反过来看React的class写法，本来就是属于面向对象的写法，状态state归状态，属性prop归属性，方法归方法，想获取什么内容，通过this直接获取，更接近于JavaScript编程，相对来说比较好理解。

### 组件改造

针对Vue的反面向对象，我们可以更改其写法，通过语法糖的形式，将其我们自己的写法编译成Vue需要的写法。

#### vue-class-component

[vue-class-component](https://www.npmjs.com/package/vue-class-component) 是Vue英文官网推荐的一个包，可以以class的模式写vue组件，它带来了很多便利：

1. methods，钩子都可以直接写作class的方法
2. computed属性可以直接通过get来获得
3. 初始化data可以声明为class的属性
4. 其他的都可以放到Component装饰器里

#### vue-property-decorator

[vue-property-decorator](https://www.npmjs.com/package/vue-property-decorator) 这个包完全依赖于vue-class-component，提供了多个装饰器，辅助完成prop、watch、model等属性的声明。

#### 编译准备

由于使用的是装饰器语法糖，我们需要在我们webpack的babel编译器中对齐进行支持。

首先是class语法支持，针对babel6及更低的版本，需要配置babel的plugin中添加class语法支持插件`babel-plugin-transform-class-properties`，针对babel7，需要使用插件`@babel/plugin-proposal-class-properties`对class进行语法转换。

然后是装饰器语法支持，针对babel6及更低的版本，需要配置babel的plugin中添加装饰器语法支持插件`babel-plugin-transform-decorators-legacy`，针对babel7，需要使用插件`@babel/plugin-proposal-decorators`对装饰器进行语法转换。

针对bable6，配置.babelrc如下

```json
{
    "presets": ["env", "stage-1"],
    "plugins": [
      "transform-runtime",
      "syntax-dynamic-import",
      "transform-class-properties",  // 新增class语法支持
      "transform-decorators-legacy" // 新增装饰器语法支持
    ]
}
```

对于bable7，官方推荐直接使用`@vue/app`preset，该预设包含了`@babel/plugin-proposal-class-properties`和`@babel/plugin-proposal-decorators`两个插件，另外还包含了动态分割加载chunks支持`@babel/plugin-syntax-dynamic-import`，同时也包含了`@babel/env`preset，.babelrc配置如下：

```json
{
  "presets": [
      ["@vue/app", {
          "loose": true,
          "decoratorsLegacy": true
      }]
  ]
}
```

#### 重写组件

编译插件准备好之后，我们对上面的Vue组件进行改写，代码如下

```html
<template>
  <div>
    <h1>{{title}}</h1>
    <span>{{text}}<span>
    <button @click="setText">按钮</button>
  </div>
</template>

<script>
import { Vue, Component, Watch, Prop } from 'vue-property-decorator';

@Component
export default class Demo extends Vue {

  text = 'hello world';

  @Prop({type: String, default: 'Vue Demo'}) title;

  @Watch('title')
  titleChange(newTitle, oldTitle) {
    console.log(`标题从 ${oldTile} 变为了 ${newTitle}`)
  }

  setText(e) {
    this.text = '点击了按钮';
  }
}
</script>
```

到此为止，我们的组件改写完毕，相对先前的“写配置”的写法，看起来相对来说要好理解一些吧。

注意：Vue的class的写法的methods还是没办法使用箭头函数进行的，详细原因这里就不展开，大概就是因为Vue内部挂载函数的方式的原因。

## 视图开发

### 特性对比

针对视图的开发，Vue推崇html、js、css分离的写法，React推崇all-in-js，所有都在js中进行写法。

当然各有各的好处，如Vue将其进行分离，代码易读性较好，但是在html中无法完美的展示JavaScript的编程能力，而对于React的jsx写法，因为有JavaScript的编程语法支持，让我们更灵活的完成视图开发。

对于这类不灵活的情况，Vue也对jsx进行了支持，只需要在babel中添加插件`babel-plugin-transform-vue-jsx`、`babel-plugin-syntax-jsx`、`babel-helper-vue-jsx-merge-props`（babel6，对于babel7，官方推荐的`@vue/app`预设中已包含了jsx的转化插件），我们就可以像React一样，在组件中声明render函数并返回jsx对象，如下我们对上一节的组件进行改造：

### 组件改造

```html
<script>
import { Vue, Component, Watch, Prop } from 'vue-property-decorator';

@Component
export default class Demo extends Vue {

  title = 'hello world';

  @Prop({type: String, default: 'Vue Demo'}) title;

  @Watch('title')
  titleChange(newTitle, oldTitle) {
    console.log(`标题从 ${oldTile} 变为了 ${newTitle}`)
  }

  setText(e) {
    this.text = '点击了按钮';
  }

  render() {
    const { title, text } = this;
    return <div>
      <h1>{title}</h1>
      <span>{text}<span>
      <button onClick={this.setText}>按钮<button>
    </div>
  }
}
</script>
```

### Vue的jsx使用注意点

写到这里，也基本上发现其写法已经与React的class写法雷同了。那么Vue的jsx和React的jsx有什么不同呢。

在React的jsx语法需要React支持，也就是说，在你使用jsx的模块中，必须引进React。

而Vue的jsx语法需要Vue的createElement支持，也就是说在你的jsx语法的作用域当中，必须存在变量`h`，变量`h`为`createElement`的别名，这是Vue生态系统中的一个通用惯例，在render中h变量由编译器自动注入到作用域中，自动注入详情见[plugin-transform-vue-jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)，如果没有变量h，需要从组件中获取并声明，代码如下:

```js
const h = this.$createElement;
```

这里借助官方的一个例子，基本包含了所有Vue的jsx常用语法，如下：

```js
// ...
render (h) {
  return (
    <div
      // normal attributes or component props.
      id="foo"
      // DOM properties are prefixed with `domProps`
      domPropsInnerHTML="bar"
      // event listeners are prefixed with `on` or `nativeOn`
      onClick={this.clickHandler}
      nativeOnClick={this.nativeClickHandler}
      // other special top-level properties
      class={{ foo: true, bar: false }}
      style={{ color: 'red', fontSize: '14px' }}
      key="key"
      ref="ref"
      // assign the `ref` is used on elements/components with v-for
      refInFor
      slot="slot">
    </div>
  )
}
```

但是，Vue的jsx语法无法支持Vue的内建指令，唯一的例外是v-show，该指令可以使用v-show={value}的语法。大多数指令都可以用编程方式实现，比如`v-if`就是一个三元表达式，`v-for`就是一个`array.map()`等。 

如果是自定义指令，可以使用`v-name={value}`语法，但是该语法不支持指令的参数`arguments`和修饰器`modifier`。有以下两个解决方法：

* 将所有内容以一个对象传入，如：`v-name={{ value, modifier: true }}`
* 使用原生的vnode指令数据格式，如：

```js
const directives = [
  { name: 'my-dir', value: 123, modifiers: { abc: true } }
]

return <div {...{ directives }}/>
```

那么，我们什么时候使用jsx，什么时候template呢？很明显，面对那么复杂多变的视图渲染，我们使用jsx语法更能得心应手，面对简单的视图，我们使用template能开发得更快。

## 状态管理

### 特性对比

针对状态管理，Vue的Vuex和React的Redux很雷同，都是Flow数据流。

对于React来说，state需要通过mapStateToProps将state传入到组件的props中，action需要通过mapDispatchToProps将action注入到组件的props中，然后在组件的props中获取并执行。

而在Vue中，store在组件的$store中，可以直接`this.$store.dispatch(actionType)`来分发action，属性也可以通过mapState，或者mapGetter把state或者getter挂载到组件的computed下，更粗暴的可以直接`this.$store.state`或者`this.$store.getter`获取，非常方便。

### 组件改造

我们为了更贴切于es6的class写法，更好的配合`vue-class-component`，我们需要通过其他的方式将store的数据注入到组件中。

#### vuex-class

[vuex-class](https://www.npmjs.com/package/vuex-class)，这个包的出现，就是为了更好的讲Vuex与class方式的Vue组件连接起来。

如下，我们声明一个store

```js
import Vuex from 'vuex';

const store = new Vuex.Store({
  modules: {
    foo: {
      namespaced: true,
      state: {
        text: 'hello world',
      },
      actions: {
        setTextAction: ({commit}, newText) => {
          commit('setText', newText);
        }
      },
      mutations: {
        setText: (state, newText) => {
          state.text = newText;
        } 
      }
    }
  }
})
```

针对这个store，我们改写我们上一章节的组件

```html
<template>
  <div>
    <h1>{{title}}</h1>
    <span>{{text}}<span>
    <button @click="setText">按钮</button>
  </div>
</template>

<script>
import { Vue, Component, Watch, Prop } from 'vue-property-decorator';
import { namespace } from 'vuex-class';

const fooModule = namespace('foo');

@Component
export default class Demo extends Vue {

  @fooModule.State('text') text;
  @fooModule.Action('setTextAction') setTextAction;

  @Prop({type: String, default: 'Vue Demo'}) title;

  @Watch('title')
  titleChange(newTitle, oldTitle) {
    console.log(`标题从 ${oldTile} 变为了 ${newTitle}`)
  }

  setText(e) {
    this.setTextAction('点击了按钮');
  }
}
</script>
```

这里可以发现，store声明了一个foo模块，然后在使用的时候从store中取出了foo模块，然后使用装饰器的形式将state和action注入到组件中，我们就可以省去dispatch的代码，让语法糖帮我们dispatch。这样的代码，看起来更贴切与面向对象。。。好吧，我承认这个代码越写越像Java了。

然而，之前的我并不是使用Redux开发React的，而是Mobx，所以这种 dispatch -> action -> matation -> state 的形式对我来说也不是很爽，我还是更喜欢把状态管理也以class的形式去编写，这个时候我又找了另外一个包(vuex-module-decorators)[https://www.npmjs.com/package/vuex-module-decorators]来改写我的store.module。

下面我们改写上面的store:

```js
import Vuex from 'vuex';
import { Module, VuexModule,  Mutation, Action } from 'vuex-module-decorators';
 
@Module
class foo extends VuexModule {
  text = 'hello world'
 
  @Mutation
  setText(text) {
    this.text = text;
  }
 
  @Action({ commit: 'setText' })
  setTextAction(text) {
    return text;
  }
}

const store = new Vuex.Store({
  modules: {
    foo: foo
})
export default store;
```

这样，我们的项目准备基本上完毕了，把Vue组件和Vuex状态管理以class的形式来编写。大概是我觉得es5的写法显得不太优雅吧，没有es6的写法那么高端。

## 结束

class语法和装饰器decorators语法都是ES6的提案，都带给了前端不一样的编程体验，大概也是前端的一个比较大的革命吧，我们应该拥抱这样的革命变化。
