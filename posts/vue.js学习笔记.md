---
title: vue.js学习笔记(一)
date: 2018-03-28 19:09:16
tags:
- vue.js
- 前端
- javascript
---

### 指令
指令(Directives)是带有`v-`前缀的特殊属性。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于DOM。
#### v-if条件判断
```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
`<template>`元素当做不可见的包裹元素

```html
<div v-if="type === 'A'">A</div>
<div v-else-if="type === 'B'">B</div> <!-- vue2.1以后才有 v-else-if指令 -->
<div v-else>Not A/B/C</div>
```
v-if是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

#### v-show显示判断
```html
<h1 v-show="ok">Hello!</h1>
```
与`v-if`不同，v-show只是进行CSS的变换。

#### v-bind属性绑定
```html
<a v-bind:href="url">...</a>
```
绑定url到href，当url有变化可响应dom重渲染，可缩写成
```html
<a :href="url">...</a>
```

#### v-on事件绑定
```html
<a v-on:click="doSomething">...</a>
```
绑定doSomething函数到a标签的点击事件，可缩写成
```html
<a @click="doSomething">...</a>
```
#### v-for循环
可循环渲染动态选项，数组形式
```html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
```
与react不同，react需要使用map方法遍历返回组件，vue直接把循环挂在标签属性上。

对象形式
```js
data: {
  options: {
    jock: '佐客',
    tom: '汤姆',
    miko: '咪口'
  }
}
```
```html
<select v-model="selected">
  <option v-for="(value, key) in options" v-bind:value="option.key">
    {{ option.value }}
  </option>
</select>
```

### 修饰符
修饰符(Modifiers)是以半角句号`.`指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。例如，`.prevent`修饰符告诉`v-on`指令对于触发的事件调用 `event.preventDefault()`：
```html
<form v-on:submit.prevent="onSubmit">...</form>
```
#### 事件修饰符
* `.prevent` => `event.preventDefault()`
* `.stop` => `event.stopPropagation()`
* `.capture` 使用事件捕获
* `.self` 只当在 event.target 是当前元素自身时触发处理函数，即事件不是从内部元素触发的
* `.once` 事件只触发一次

#### 按键修饰符
```html
<!-- 只有在 `keyCode` 是 13 时调用 `vm.submit()` -->
<input v-on:keyup.13="submit">

<!-- 同上 -->
<input v-on:keyup.enter="submit">

<!-- 缩写语法 -->
<input @keyup.enter="submit">
```
全部的按键别名：

* .enter
* .tab
* .delete (捕获“删除”和“退格”键)
* .esc
* .space
* .up
* .down
* .left
* .right

#### 系统修饰键
* .ctrl
* .alt
* .shift
* .meta
在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。

#### .exact 修饰符
.exact 修饰符允许你控制由精确的系统修饰符组合触发的事件。
```html
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

#### 鼠标按钮修饰符
* .left
* .right
* .middle
这些修饰符会限制处理函数仅响应特定的鼠标按钮。

### 计算属性`computed`
基础例子如下
```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```
computed下的`reversedMessage`方法依赖data下的变量message，当message发生变量，reversedMessage会重新计算结果，与mobx中的computed的作用相同。
以上例子也可通过`method`属性完成，如下
```html
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```js
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```
不同的是计算属性是基于它们的依赖进行缓存的。

计算属性`computed`也可同时设置setter和getter方法取代直接写funciton，如下：
```js
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```
### 侦听器`watch`
```js
new Vue({
  data: {
    watchData: ''
  },
  watch: {
    watchData: function (newData, oldData) {
      // doSomething
    }
  }
})
```

### Class与Style
#### class的对象形式，与classnames模块类似
```html
<div v-bind:class="{ active: isActive }"></div>
```

#### class数组形式
```html
<div v-bind:class="[activeClass, errorClass]"></div>
```
```js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

#### class对象与数组混合使用
```html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

#### 在组件上使用
```js
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```
```html
<my-component class="baz boo"></my-component>
```
结果为：
```html
<p class="foo bar baz boo">Hi</p>
```

#### style对象形式
```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```js
data: {
  activeColor: 'red',
  fontSize: 30
}
```

#### style数组形式
```html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

### 表单输入绑定
`v-model`指令在表单`input`及`textarea`元素上创建双向数据绑定。它会根据控件类型自动选取正确的方法来更新元素。v-model会忽略所有表单元素的value、checked、selected特性的初始值而总是将Vue实例的数据作为数据来源。你应该通过JavaScript在组件的data选项中声明初始值。
如:
```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```
多行文本
```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```
#### .lazy
在默认情况下，v-model 在每次 input 事件触发后将输入框的值与数据进行同步。你可以添加 lazy 修饰符，从而转变为使用 change 事件进行同步：
```html
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" >
```

#### .number
如果想自动将用户的输入值转为数值类型，可以给 v-model 添加 number 修饰符：
```html
<input v-model.number="age" type="number">
```

#### .trim
如果要自动过滤用户输入的首尾空白字符，可以给 v-model 添加 trim 修饰符：
```html
<input v-model.trim="msg">
```

### 组件
组件 (Component) 是Vue.js最强大的功能之一。组件可以扩展HTML元素，封装可重用的代码。在较高层面上，组件是自定义元素，Vue.js的编译器为它添加特殊功能。在有些情况下，组件也可以表现为用`is`特性进行了扩展的原生 HTML 元素。
请注意，对于自定义标签的命名Vue.js不强制遵循[W3C规则](https://www.w3.org/TR/custom-elements/#concepts) (小写，并且包含一个短杠)，尽管这被认为是最佳实践。

全局注册组件
```js
Vue.component('my-component', {
  // 选项
})
```

局部注册组件
```js
var Child = {
  template: '<div>A custom component!</div>'
}

new Vue({
  // ...
  components: {
    // <my-component> 将只在父组件模板中可用
    'my-component': Child
  }
})
```
由于html元素之间不能随意嵌套，如table、ul、ol，所以在自定义组件中使用这些受限制的元素时会导致一些问题，例如：
```html
<table>
  <my-row>...</my-row>
</table>
```
自定义组件<my-row>会被当作无效的内容，因此会导致错误的渲染结果。变通的方案是使用特殊的`is`特性：
```html
<table>
  <tr is="my-row"></tr>
</table>
```
但是如果代码来自以下字符串模板，则没有这个限制：
* &lt;script type="text/x-template"&gt;
* JavaScript 内联模板字符串
* .vue 组件

#### `data`必须是函数
声明的组件data必须为一个函数并返回响应数据，否则会出现警告并停止运行，声明比如类似如下：
```js
var data = { counter: 0 }

Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  // 技术上 data 的确是一个函数了，因此 Vue 不会警告，
  // 但是我们却给每个组件实例返回了同一个对象的引用
  data: function () {
    return {
      counter: 0
    }
  }
})

new Vue({
  el: '#example-2'
})
```

#### Prop
与react类似，prop为传入组件的参数，属于单向数据流，如果是绑定了父组件的data，变化会引起DOM重渲染，声明组件时候需要声明预期的props，如下：
```js
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 也可以在模板中使用
  // 同样也可以在 vm 实例中通过 this.message 来使用
  template: '<span>{{ message }}</span>'
})
```

由于HTML不区分大小写，声明prop的时候我们需要声明为`camelCase`驼峰命名形式，但是传入的时候必须写成`kebab-case`短横线分隔式命名。如下：
```js

Vue.component('child', {
  // 在 JavaScript 中使用 camelCase
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```
```html
<!-- 在 HTML 中使用 kebab-case -->
<child my-message="hello!"></child>
```

动态Prop
可以用 v-bind 来动态地将 prop 绑定到父组件的数据。每当父组件的数据变化时，该变化也会传导给子组件：
```html
<div id="prop-example-2">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```
```js
new Vue({
  el: '#prop-example-2',
  data: {
    parentMsg: 'Message from parent'
  }
})
```

prop声明时可以指定特定规则，规范prop的格式，如：
```js
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 指允许任何类型)
    propA: Number,
    // 可能是多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数值且有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```
type也可以是一个自定义构造器函数，使用`instanceof`检测。

#### 动态组件
通过使用保留的&lt;component&gt; 元素，并对其`is`特性进行动态绑定，你可以在同一个挂载点动态切换多个组件：
```js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```
```html
<component v-bind:is="currentView">
  <!-- 组件在 vm.currentview 变化时改变！ -->
</component>
```
也可以直接绑定到组件对象上：
```js
var Home = {
  template: '<p>Welcome home!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

#### 插槽&lt;slot&gt;
&lt;slot&gt;类似于react中的props.children，可以替换输出父组件中传给子组件标签内的内容，如下：
假定 my-component 组件有如下模板：
```html
<div>
  <h2>我是子组件的标题</h2>
  <slot>
    只有在没有要分发的内容时才会显示。
  </slot>
</div>
```
父组件模板：
```html
<div>
  <h1>我是父组件的标题</h1>
  <my-component>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </my-component>
</div>
```
渲染结果：
```html
<div>
  <h1>我是父组件的标题</h1>
  <div>
    <h2>我是子组件的标题</h2>
    <p>这是一些初始内容</p>
    <p>这是更多的初始内容</p>
  </div>
</div>
```

&lt;slot&gt;元素可以用一个特殊的特性 name 来进一步配置如何分发内容。多个插槽可以有不同的名字。具名插槽将匹配内容片段中有对应slot特性的元素。
例如，假定我们有一个 app-layout 组件，它的模板为：
```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
父组件模板：
```html
<app-layout>
  <h1 slot="header">这里可能是一个页面标题</h1>

  <p>主要内容的一个段落。</p>
  <p>另一个主要段落。</p>

  <p slot="footer">这里有一些联系信息</p>
</app-layout>
```
渲染结果为：
```html
<div class="container">
  <header>
    <h1>这里可能是一个页面标题</h1>
  </header>
  <main>
    <p>主要内容的一个段落。</p>
    <p>另一个主要段落。</p>
  </main>
  <footer>
    <p>这里有一些联系信息</p>
  </footer>
</div>
```
在设计组合使用的组件时，内容分发 API 是非常有用的机制。

#### 子组件引用
但是有时仍然需要在JavaScript中直接访问子组件。与react类似，可以使用`ref`为子组件指定一个引用ID。例如：
```html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```
```js
var parent = new Vue({ el: '#parent' })
// 访问子组件实例
var child = parent.$refs.profile
```
当ref和v-for一起使用时，获取到的引用会是一个数组，包含和循环数据源对应的子组件。

### 自定义事件
#### 使用 v-on 绑定自定义事件
* 使用 $on(eventName) 监听事件
* 使用 $emit(eventName, optionalPayload) 触发事件

例子如下：
```html
<div id="message-event-example" class="demo">
  <p v-for="msg in messages">{{ msg }}</p>
  <button-message v-on:message="handleMessage"></button-message>
</div>
```
```js
Vue.component('button-message', {
  template: `<div>
    <input type="text" v-model="message" />
    <button v-on:click="handleSendMessage">Send</button>
  </div>`,
  data: function () {
    return {
      message: 'test message'
    }
  },
  methods: {
    handleSendMessage: function () {
      this.$emit('message', { message: this.message })
    }
  }
})

new Vue({
  el: '#message-event-example',
  data: {
    messages: []
  },
  methods: {
    handleMessage: function (payload) {
      this.messages.push(payload.message)
    }
  }
})
```

#### 给组件绑定原生事件
可以使用v-on的修饰符`.native`。例如：
```html
<my-component v-on:click.native="doTheThing"></my-component>
```

#### .sync修饰符
该修饰符可以让一个prop变成"双向绑定"，如下
```html
<comp :foo.sync="bar"></comp>
```
当comp内修改了foo变量时，父组件的foo也会同时被修改，该修饰符Vue2.0后被移除，在Vue2.3又被引入，只是这次它只是作为一个编译时的语法糖存在。它会被扩展为一个自动更新父组件属性的 v-on 监听器。如上例子会被扩展成：
```html
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```
当子组件需要更新 foo 的值时，它需要显式地触发一个更新事件：
```js
this.$emit('update:foo', newValue)
```

#### 自定义组件的`v-model`
默认情况下，一个组件的 v-model 会使用 value prop 和 input 事件。但是诸如单选框、复选框之类的输入类型可能把 value 用作了别的目的。model 选项可以避免这样的冲突：
```js
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean,
    // 这样就允许拿 `value` 这个 prop 做其它事了
    value: String
  },
  // ...
})
```
```html
<my-checkbox v-model="foo" value="some value"></my-checkbox>
```
上述代码等价于：
```html
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="some value">
</my-checkbox>
```
