#### 介绍一下Vue的响应式系统

#### computed与watch的区别

#### 介绍一下Vue的生命周期

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