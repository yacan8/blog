#### 介绍一下Vue的响应式系统

Vue为MVVM框架，当数据模型data变化时，页面视图会得到响应更新，其原理对data的getter/setter方法进行拦截（Object.defineProperty或者Proxy），利用发布订阅的设计模式，在getter方法中进行订阅，在setter方法中发布通知，让所有订阅者完成响应。

在响应式系统中，Vue会为数据模型data的每一个属性新建一个订阅中心作为发布者，而监听器watch、计算属性computed、视图渲染template/render三个角色同时作为订阅者，对于监听器watch，会直接订阅观察监听的属性，对于计算属性computed和视图渲染template/render，如果内部执行获取了data的某个属性，就会执行该属性的getter方法，然后自动完成对该属性的订阅，当属性被修改时，就会执行该属性的setter方法，从而完成该属性的发布通知，通知所有订阅者进行更新。

#### computed与watch的区别

计算属性computed和监听器watch都可以观察属性的变化从而做出响应，不同的是：

计算属性computed更多是作为缓存功能的观察者，它可以将一个或者多个data的属性进行复杂的计算生成一个新的值，提供给渲染函数使用，当依赖的属性变化时，computed不会立即重新计算生成新的值，而是先标记为脏数据，当下次computed被获取时候，才会进行重新计算并返回。

而监听器watch并不具备缓存性，监听器watch提供一个监听函数，当监听的属性发生变化时，会立即执行该函数。

#### 介绍一下Vue的生命周期

`beforeCreate`：是new Vue()之后触发的第一个钩子，在当前阶段data、methods、computed以及watch上的数据和方法都不能被访问。

`created`：在实例创建完成后发生，当前阶段已经完成了数据观测，也就是可以使用数据，更改数据，在这里更改数据不会触发updated函数。可以做一些初始数据的获取，在当前阶段无法与Dom进行交互，如果非要想，可以通过vm.$nextTick来访问Dom。

`beforeMount`：发生在挂载之前，在这之前template模板已导入渲染函数编译。而当前阶段虚拟Dom已经创建完成，即将开始渲染。在此时也可以对数据进行更改，不会触发updated。

`mounted`：在挂载完成后发生，在当前阶段，真实的Dom挂载完毕，数据完成双向绑定，可以访问到Dom节点，使用$refs属性对Dom进行操作。

`beforeUpdate`：发生在更新之前，也就是响应式数据发生更新，虚拟dom重新渲染之前被触发，你可以在当前阶段进行更改数据，不会造成重渲染。

`updated`：发生在更新完成之后，当前阶段组件Dom已完成更新。要注意的是避免在此期间更改数据，因为这可能会导致无限循环的更新。

`beforeDestroy`：发生在实例销毁之前，在当前阶段实例完全可以被使用，我们可以在这时进行善后收尾工作，比如清除计时器。

`destroyed`：发生在实例销毁之后，这个时候只剩下了dom空壳。组件已被拆解，数据绑定被卸除，监听被移出，子实例也统统被销毁。

#### 为什么data必须是一个函数

#### 组件之间是怎么通信的

#### Vue事件绑定原理说一下

#### 说一下什么是Virtual DOM

#### 介绍一下Vue中的Diff算法

#### key属性的作用是什么

#### Vue的模板编译，有了解过吗

#### 说说Vue2.0和Vue3.0有什么区别

1. 重构响应式系统，使用Proxy替换Object.defineProperty，使用Proxy优势：
  * 可直接监听数组类型的数据变化
  * 监听的目标为对象本身，不需要像Object.defineProperty一样遍历每个属性，有一定的性能提升
  * 可拦截apply、ownKeys、has等13种方法，而Object.defineProperty不行
  * 直接实现对象属性的新增/删除
2. 新增Composition API，更好的逻辑复用和代码组织
3. 重构 Virtual DOM
  * 模板编译时的优化，将一些静态节点编译成常量
  * slot优化，将slot编译为lazy函数，将slot的渲染的决定权交给子组件
  * 模板中内联事件的提取并重用（原本每次渲染都重新生成内联函数）

4. 代码结构调整，更便于Tree shaking，使得体积更小
5. 使用Typescript替换Flow

#### 为什么要新增Composition API，它能解决什么问题

Vue2.0中，随着功能的增加，组件变得越来越复杂，越来越难维护，而难以维护的根本原因是Vue的API设计迫使开发者使用watch，computed，methods选项组织代码，而不是实际的业务逻辑。

另外Vue2.0缺少一种较为简洁的低成本的机制来完成逻辑复用，虽然可以minxis完成逻辑复用，但是当mixin变多的时候，会使得难以找到对应的data、computed或者method来源于哪个mixin，使得类型推断难以进行。

所以Composition API的出现，主要是也是为了解决Option API带来的问题，第一个是代码组织问题，Compostion API可以让开发者根据业务逻辑组织自己的代码，让代码具备更好的可读性和可扩展性，也就是说当下一个开发者接触这一段不是他自己写的代码时，他可以更好的利用代码的组织反推出实际的业务逻辑，或者根据业务逻辑更好的理解代码。

第二个是实现代码的逻辑提取与复用，当然mixin也可以实现逻辑提取与复用，但是像前面所说的，多个mixin作用在同一个组件时，很难看出property是来源于哪个mixin，来源不清楚，另外，多个mixin的property存在变量命名冲突的风险。而Composition API刚好解决了这两个问题。

#### 都说Composition API与React Hook很像，说说区别

从React Hook的实现角度看，React Hook是根据useState调用的顺序来确定下一次重渲染时的state是来源于哪个useState，所以出现了以下限制

* 不能在循环、条件、嵌套函数中调用Hook
* 必须确保总是在你的React函数的顶层调用Hook
* useEffect、useMemo等函数必须手动确定依赖关系

而Composition API是基于Vue的响应式系统实现的，与React Hook的相比

* 声明在setup函数内，一次组件实例化只调用一次setup，而React Hook每次重渲染都需要调用Hook，使得React的GC比Vue更有压力，性能也相对于Vue来说也较慢
* Compositon API的调用不需要顾虑调用顺序，也可以在循环、条件、嵌套函数中使用
* 响应式系统自动实现了依赖收集，进而组件的部分的性能优化由Vue内部自己完成，而React Hook需要手动传入依赖，而且必须必须保证依赖的顺序，让useEffect、useMemo等函数正确的捕获依赖变量，否则会由于依赖不正确使得组件性能下降。

虽然Compositon API看起来比React Hook好用，但是其设计思想也是借鉴React Hook的。

#### SSR有了解吗？原理是什么？


## 参考文献

[「面试题」20+Vue面试题整理](https://juejin.im/post/6844904084374290446)