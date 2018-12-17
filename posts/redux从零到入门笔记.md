---
title: redux从零到入门笔记
date: 2018-01-29 18:34:25
tags:
- react
- redux
---

## 为什么需要redux
学过react的都知道，react用`state`和`props`控制组件的渲染情况，而对于JavaScript单页面日趋复杂的今天，JavaScript需要管理越来越多的state，而这些state包括着各种乱七八糟途径来的数据。甚至有的应用的state会关系到另一个组件的状态。所以为了方便对这些state的管理以及对state变化的可控性。这个时候Redux这个东西就出来了，它可以让state的变化变得可预测。
## Redux的基本概念
什么是redux？这里非权威的解释：就是一个应用的state管理库，甚至可以说是前端数据库。更包括的是管理数据。
### state
state是整个应用的数据，本质上是一个普通对象。
state决定了整个应用的组件如何渲染，渲染的结果是什么。可以说，State是应用的灵魂，组件是应用的肉体。
所以，在项目开发初期，设计一份健壮灵活的State尤其重要，对后续的开发有很大的帮助。
但是，并不是所有的数据都需要保存到state中，有些属于组件的数据是完全可以留给组件自身去维护的。
### action
数据state已经有了，那么我们是如何实现管理这些state中的数据的呢？那就是action，什么是action？按字面意思解释就是动作，也可以理解成，一个可能！改变state的动作包装。就这么简单。。。。
只有当某一个动作发生的时候才能够触发这个state去改变，那么，触发state变化的原因那么多，比如这里的我们的点击事件，还有网络请求，页面进入，鼠标移入。。。所以action的出现，就是为了把这些操作所产生或者改变的数据从应用传到store中的有效载荷。 需要说明的是，action是state的唯一来源。它本质上就是一个JavaScript对象，但是约定的包含`type`属性，可以理解成每个人都要有名字一般。除了type属性，别的属性，都可以.
那么这么多action一个个手动创建必然不现实，一般我们会写好`actionCreator`，即action的创建函数。调用`actionCreator`，给你返回一个action。这里我们可以使用 [redux-actions](https://www.npmjs.com/package/redux-actions)，嗯呢，我们下文有介绍。
比如有一个counter数量加减应用，我们就有两个action，一个`decrement`，一个`increment`。 所以这里的action creator写成如下：
```js
export function decrement() {
    return{
        type:DECREMENT_COUNTER
    }
}

export function increment(){
    return{
        type:INCREMENT_COUNTER
    }
}
```
那么，当action创建完成了之后呢，我们怎么触发这些action呢，这时我们是要利用`dispatch`，比如我们执行count增减减少动作。
```js
export function incrementIfOdd(){
    return(dispatch,getState)=>{
        const {counter} = getState();
        if(counter%2==0) {
            return;
        }
        dispatch(increment());
    }
}

export function incrementAsync() {
    return dispatch => {
        setTimeout(() => {
            dispatch(increment());
        }, 1000);
    };
}
```
为了减少样板代码，我们使用单独的模块或文件来定义 action type 常量
```js
export const INCREMENT_COUNTER = 'INCREMENT_COUNTER';
export const DECREMENT_COUNTER = 'DECREMENT_COUNTER';
```
这么做不是必须的，在大型应用中把它们显式地定义成常量还是利大于弊的。
### reducer
既然这个可能改变state的动作已经包装好了，那么我们怎么去判断并且对state做相应的改变呢？对，这就是reducer干的事情了。
`reducer`是state最终格式的确定。它是一个纯函数，也就是说，只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。
`reducer`对传入的action进行判断，然后返回一个通过判断后的state，这就是reducer的全部职责。如我们的counter应用：
```js
import {INCREMENT_COUNTER,DECREMENT_COUNTER} from '../actions';

export default function counter(state = 0, action) {
    switch (action.type){
        case INCREMENT_COUNTER:
            return state+1;
        case DECREMENT_COUNTER:
            return state-1;
        default:
            return state;
    }
}
```
这里我们就是对增和减两个之前在action定义好的常量做了处理。
对于一个比较大一点的应用来说，我们是需要将reducer拆分的，最后通过redux提供的combineReducers方法组合到一起。 如此项目上的：
```js
const rootReducer = combineReducers({
    counter
});
export default rootReducer;
```
每个`reducer`只负责管理全局`state`中它负责的一部分。每个`reducer`的`state`参数都不同，分别对应它管理的那部分`state`数据。`combineReducers()`所做的只是生成一个函数，这个函数来调用你的一系列`reducer`，每个`reducer`根据它们的`key`来筛选出`state`中的一部分数据并处理， 然后这个生成的函数再将所有`reducer`的结果合并成一个大的对象。
### store
store是对之前说到一个联系和管理。具有如下职责
* 维持应用的`state`；
* 提供`getState()`方法获取 state
* 提供`dispatch(action)`方法更新 state；
* 通过`subscribe(listener)`注册监听器;
* 通过`subscribe(listener)`返回的函数注销监听器。
强调一下 Redux 应用只有一个单一的`store`。当需要拆分数据处理逻辑时，你应该使用`reducer`组合,而不是创建多个`store`。`store`的创建通过`redux`的`createStore`方法创建，这个方法还需要传入`reducer`，很容易理解：毕竟我需要`dispatch`一个`action`来改变`state`嘛。 应用一般会有一个初始化的`state`，所以可选为第二个参数，这个参数通常是有服务端提供的，传说中的`Universal`渲染。后面会说。。。 第三个参数一般是需要使用的中间件，通过applyMiddleware传入。
说了这么多，`action`，`store`，`actionCreator`，`reducer`关系就是这么如下的简单明了：

![redux](./redux.png)

## 结合react-redux的使用
`react-redux`，`redux`和`react`的桥梁工具。
`react-redux`将组建分成了两大类，UI组建`component`和容器组建`container`。 简单的说，UI组建负责美的呈现，容器组件负责来帮你盛着，给你"力量"。
UI 组件有以下几个特征：
* 只负责 UI 的呈现，不带有任何业务逻辑
* 没有状态（即不使用this.state这个变量）
* 所有数据都由参数（this.props）提供
* 不使用任何 Redux 的 API
如：
```js
export default class Counter extends Component{
    render(){
        const { counter, increment, decrement, incrementIfOdd, incrementAsync } = this.props;
        return(
            <p>
                Clicked:{counter} times
                <button onClick={increment}>+</button>
                <button onClick={decrement}>-</button>
                <button onClick={incrementIfOdd}>increment if Odd</button>
                <button onClick={incrementAsync}>increment async</button>
            </p>
        )
    }
}
```
容器组件特性则恰恰相反：
* 负责管理数据和业务逻辑，不负责 UI 的呈现
* 带有内部状态
* 使用 Redux 的 API
```js
class App extends Component{
    render(){
        const { counter, increment, decrement, incrementIfOdd, incrementAsync } = this.props;
        return(
            <Counter
                counter={counter}
                increment={increment}
                decrement={decrement}
                incrementIfOdd={incrementIfOdd}
                incrementAsync={incrementAsync}/>
        )
    }
}

export default connect(
    state=>({ counter: state.counter }),
    ActionCreators
)(App);
```
`connect`方法接受两个参数：`mapStateToProps`和`mapDispatchToProps`。它们定义了UI组件的业务逻辑。前者负责输入逻辑，即将`state`映射到 UI 组件的参数（props）， 后者负责输出逻辑，即将用户对 UI 组件的操作映射成`Action`。因为作为组件，我们只要能拿到值，能发出改变值得action就可以了，所以`mapStateToProps`和`mapDispatchToProps`正是满足这个需求的。
## redux-thunk
一个比较流行的redux的action中间件，它可以让`actionCreator`暂时不返回`action`对象，而是返回一个函数，函数传递两个参数`(dispatch, getState)`，在函数体内进行业务逻辑的封装，比如异步操作，我们至少需要触发两个`action`，这时候我们可以通过`redux-thunk`将这两个`action`封装在一起，如下：
```js
const fetchDataAction = (querys) => (dispatch, getState) => {
    const setLoading = createAction('SET_LOADING');
    dispatch(setLoading(true)); // 设置加载中。。。
    return fetch(`${url}?${querys}`).then(r => r.json()).then(res => {
        dispatch(setLoading(false)); // 设置取消加载中。。。
        dispatch(createAction('DATA_DO_SOMETHIN')(res))
    })
}
```
这里我们的`createCreator`返回的是一个`fetch`对象，我们下文会介绍，我们通过`dispatch`触发改`action`
```js
dispatch(fetchDataAction(querys))
```
在请求数据之前，通过`redux-thunk`我们可以先触发加载中的`action`，等请求数据结束之后我们可以在次触发`action`，使得加载中状态取消，并处理请求结果。
## redux-promise
既然说到了异步`action`，我们可以使用`redux-promise`，它可以让`actionCreator`返回一个`Promise`对象。
第一种做法，我们可以参考`redux-thunk`的部分。
第二种做法，`action`对象的`payload`属性（相当于我们的diy参数，action里面携带的其他参数）是一个`Promise`对象。这需要从`redux-actions`模块引入`createAction`方法，并且写法也要变成下面这样。
```js
import { createAction } from 'redux-actions';
class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // 发出异步 Action
    dispatch(createAction(
      'FETCH_DATA',
      fetch(`url`).then(res => res.json())
    ));
  }
```
其实`redux-actions`的`createAction`的源码是拿到fetch对象的payload结果之后又触发了一次`action`。
## redux-actions
当我们的在开发大型应用的时候，对于大量的`action`，我们的`reducer`需要些大量的swich来对`action.type`进行判断。`redux-actions`可以简化这一烦琐的过程，它可以是`actionCreator`，也可以用来生成`reducer`，其作用都是用来简化`action`、`reducer`。
主要函数有`createAction`、`createActions`、`handleAction`、`handleActions`、`combineActions`。
### createAction
创建`action`，参数如下
```js
import { createAction } from 'redux-actions';
createAction(
  type,  // action类型
  payloadCreator = Identity, // payload数据 具体参考Flux教程
  ?metaCreator // 具体我也没深究是啥
)
```
例子如下：
```js
export const increment = createAction('INCREMENT')
export const decrement = createAction('DECREMENT')

increment() // { type: 'INCREMENT' }
decrement() // { type: 'DECREMENT' }
increment(10) // { type: 'INCREMENT', payload: 10 }
decrement([1, 42]) // { type: 'DECREMENT', payload: [1, 42] }
```

### createActions

创建多个`action`。

```js
import { createActions } from 'redux-actions';
createActions(
  actionMap,
  ?...identityActions,
)
```
第一个参数`actionMap`为一个对象，以`action type`为键值，值value有三种形式，
* 函数，该函数参数传入的是`action`创建的时候传入的参数，返回结果会作为到生成的`action`的`payload`的value。
* 数组，长度为二，第一个值为一个函数，前面的一样，返回`payload`的值，第二个值也为一个函数，返回`meta`的值，不知道有什么用。
* 一个 `actionMap`对象，递归作用吧。
例子如下
```js
createActions({
  ADD_TODO: todo => ({ todo })
  REMOVE_TODO: [
    todo => ({ todo }), // payloa
    (todo, warn) => ({ todo, warn }) // meta
  ]
});
```
```js
const actionCreators = createActions({
  APP: {
    COUNTER: {
      INCREMENT: [
        amount => ({ amount }),
        amount => ({ key: 'value', amount })
      ],
      DECREMENT: amount => ({ amount: -amount }),
      SET: undefined // given undefined, the identity function will be used
    },
    NOTIFY: [
      (username, message) => ({ message: `${username}: ${message}` }),
      (username, message) => ({ username, message })
    ]
  }
});

expect(actionCreators.app.counter.increment(1)).to.deep.equal({
  type: 'APP/COUNTER/INCREMENT',
  payload: { amount: 1 },
  meta: { key: 'value', amount: 1 }
});
expect(actionCreators.app.counter.decrement(1)).to.deep.equal({
  type: 'APP/COUNTER/DECREMENT',
  payload: { amount: -1 }
});
expect(actionCreators.app.counter.set(100)).to.deep.equal({
  type: 'APP/COUNTER/SET',
  payload: 100
});
expect(actionCreators.app.notify('yangmillstheory', 'Hello World')).to.deep.equal({
  type: 'APP/NOTIFY',
  payload: { message: 'yangmillstheory: Hello World' },
  meta: { username: 'yangmillstheory', message: 'Hello World' }
});
```
第二个参数`identityActions`，可选参数，也是一个`action type`吧，官方例子没看懂，如下：
```js
const { actionOne, actionTwo, actionThree } = createActions({
  // function form; payload creator defined inline
  ACTION_ONE: (key, value) => ({ [key]: value }),

  // array form
  ACTION_TWO: [
    (first) => [first],             // payload
    (first, second) => ({ second }) // meta
  ],

  // trailing action type string form; payload creator is the identity
}, 'ACTION_THREE');

expect(actionOne('key', 1)).to.deep.equal({
  type: 'ACTION_ONE',
  payload: { key: 1 }
});

expect(actionTwo('first', 'second')).to.deep.equal({
  type: 'ACTION_TWO',
  payload: ['first'],
  meta: { second: 'second' }
});

expect(actionThree(3)).to.deep.equal({
  type: 'ACTION_THREE',
  payload: 3,
});
```

### handleAction
字面意思理解，处理`action`，那就是一个`reducer`，包裹返回一个`reducer`，处理一种类型的`action type`。
```js
import { handleAction } from 'redux-actions';

handleAction(
  type,  // action类型
  reducer | reducerMap = Identity
  defaultState // 默认state
)
```
当第二个参数为一个reducer处理函数时，形式如下，处理传入的`state`并返回新的`state`：
```js
handleAction('APP/COUNTER/INCREMENT', (state, action) => ({
  counter: state.counter + action.payload.amount,
}), defaultState);
```
当第二个参数为reducerMap时，也为处理`state`并返回新的`state`，只是必须传入key值为`next`和`throw`的两个函数，分别用来处理state和异常如下：
```js
handleAction('FETCH_DATA', {
  next(state, action) {...},
  throw(state, action) {...},
}, defaultState);
```
官方推荐使用`reducerMap`形式，因为与ES6的`generator`类似。
### handleActions
与`handleAction`不同，`handleActions`可以处理多个`action`，也返回一个`reducer`。
```js
import { handleActions } from 'redux-actions';

handleActions(
  reducerMap,
  defaultState
)
```
`reducerMap`以`action type`为key，value与`handleAction`的第二个参数一致，传入一个`reducer`处理函数或者一个只有`next`和`throw`两个键值的对象。
另外，键值key也可以使用`createAction`创建：
```js
import { createActions, handleActions } from 'redux-actions';

const { increment, decrement } = createActions({
  'INCREMENT': amount => ({ amount: 1 }),
  'DECREMENT': amount => ({ amount: -1 })
});

const reducer = handleActions({
  [increment](state, { payload: { amount } }) {
    return { counter: state.counter + amount }
  },
  [decrement](state, { payload: { amount } }) {
    return { counter: state.counter + amount }
  }
}, defaultState);
```
### combineActions
将多个`action`或者`actionCreator`结合起来，看起来很少用，具体例子如下：
```js
const { increment, decrement } = createActions({
  INCREMENT: amount => ({ amount }),
  DECREMENT: amount => ({ amount: -amount })
});

const reducer = handleActions({
  [combineActions(increment, decrement)](state, { payload: { amount } }) {
    return { ...state, counter: state.counter + amount };
  }
}, { counter: 10 });

expect(reducer({ counter: 5 }, increment(5))).to.deep.equal({ counter: 10 });
expect(reducer({ counter: 5 }, decrement(5))).to.deep.equal({ counter: 0 });
expect(reducer({ counter: 5 }, { type: 'NOT_TYPE', payload: 1000 })).to.equal({ counter: 5 });
expect(reducer(undefined, increment(5))).to.deep.equal({ counter: 15 });
```
`redux-actions`说到这里，大概是这样，有什么不了解看看[官方文档](https://redux-actions.js.org)吧。
## reselect
`Reselect`用来记忆`selectors`的库，我们定义的`selectors`是作为函数获取`state`的某一部分。使用记忆能力，我们可以组织不必要的衍生数据的重渲染和计算过程，由此加速了我们的应用。具体细节大概是在`mapStateToProps`的时候，讲`state`的某一部分交给`reselect`的`selectors`来管理，使用`selectors`的记忆功能让组件的`props`尽量不变化，引起不必要的渲染。
下面我们以一个todolist为例子。
当我们没有`reselect`的时候，我们是直接通过`mapStateToProps`把数据传入组件内，如下。
```js
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state, props) => {
  return {
    todolist: getVisibleTodos(state, props)
  }
}
```
这个代码有一个潜在的问题。每当`state tree`改变时，`selector`都要重新运行。当`state tree`特别大，或者`selector`计算特别耗时，那么这将带来严重的运行效率问题。为了解决这个问题，reselect为`selector`设置了缓存，只有当`selector`的输入改变时，程序才重新调用selector函数。
这时我们把`state`转化为`props`的数据交给`reselect来`处理，我们重写`mapStateToProps`。
```js
const getVisibilityFilter = state => state.todo.showStatus

const getTodos = state => state.todo.todolist

const getVisibleTodos = createSelector([getVisibilityFilter, getTodos], (visibilityFilter, todos) => {
  switch (visibilityFilter) {
    case 'SHOW_COMPLETED':
      return todos.filter(todo => todo.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(todo => !todo.completed)
    default:
      return todos
  }
})
const mapStateToProps = (state, props) => {
  const todolist = getVisibleTodos(state, props);
  return {
    todolist
  }
}
```
我们使用`createSelector`包裹起来，将组件内需要的两个`props`包裹起来，然后在返回一个获取数据的函数`getVisibleTodos`，这样返回的`todolist`就不会受到一些不必要的state的变化而变化引起冲渲染。
## 最后
总结了那么多的用法，其实也是`redux`的基本用法，然后自己写了半天的`todolist`，把上面说到的技术都用了，这是 [github地址](https://github.com/yacan8/react-redux-todo)，上面的内容如有错误，勿喷，毕竟入门级别。。。
