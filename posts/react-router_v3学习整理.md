---
title: react-router v3学习整理
date: 2017-11-18 15:21:05
tags:
- react
- react-router
---

## 简介
React Router是一个基于 React 之上的强大路由库，它可以让你向应用中快速地添加视图和数据流，同时保持页面与URL间的同步。在没有react-router的时候，我们需要对URL进行监听，当URL的hash部分（指的是 # 后的部分）变化后，根据hash来渲染不同的组件。看起来很直接，但它很快就会变得复杂起来，当我们的组件嵌套层级增加的时候，为了让我们的URL解析变得更智能，我们需要编写很多代码来实现指定URL应该渲染哪一个嵌套的UI组件分支，所以我们需要另外一个解决方案，这个时候react-router出现了。

## 路由配置

### 基本使用

路由配置是一组指令，用来告诉 router 如何匹配 URL以及匹配后如何执行代码。我们来通过一个简单的例子解释一下如何编写路由配置。

```jsx
import React from 'react'
import { Router, Route, Link } from 'react-router'

const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>
        {this.props.children}
      </div>
    )
  }
})

const About = React.createClass({
  render() {
    return <h3>About</h3>
  }
})

const Inbox = React.createClass({
  render() {
    return (
      <div>
        <h2>Inbox</h2>
        {this.props.children || "Welcome to your Inbox"}
      </div>
    )
  }
})

const Message = React.createClass({
  render() {
    return <h3>Message {this.props.params.id}</h3>
  }
})

React.render((
  <Router>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

通过上面的配置，这个应用知道如何渲染下面四个 URL：

| URL | 组件 |
| ---| --- |
| /  | App |
| /about | App -> About |
| /indox | App -> Inbox |
| /inbox/message/:id | App -> Indox -> Message |

如果我们可以将 `/inbox` 从 `/inbox/messages/:id` 中去除，并且还能够让 Message 嵌套在 App -> Inbox 中渲染，那会非常赞。绝对路径可以让我们做到这一点。

```jsx
React.render((
  <Router>
    <Route path="/" component={App}>
      <IndexRoute component={Dashboard} />
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        {/* 使用 /messages/:id 替换 messages/:id */}
        <Route path="/messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

在多层嵌套路由中使用绝对路径的能力让我们对 URL 拥有绝对的掌控。我们无需在 URL 中添加更多的层级，从而可以使用更简洁的 URL。
我们现在的 URL 对应关系如下：

| URL | 组件 |
| --- | --- |
| /   |  App |
| /about | App -> About |
| /indox |  App -> Inbox |
| /message/:id | App -> Indox -> Message |

### 参数获取

在Message组件中，我们可以通过以下方式获取路由参数中的id

```jsx
const Message = React.createClass({
  render(){
    // 只适用于 /message/:id 形式的路由参数
    // 若要获取 ?id=yourId 形式的参数，使用this.props.location.query
    const { id } = this.props.params;
    return <div>参数id：{id}</div>
  }
})
```

### 进入和离开的Hook

Route 可以定义 onEnter 和 onLeave 两个 hook ，这些hook会在页面跳转确认时触发一次。这些 hook 对于一些情况非常的有用，例如权限验证或者在路由跳转前将一些数据持久化保存起来。

在路由跳转过程中，onLeave hook 会在所有将离开的路由中触发，从最下层的子路由开始直到最外层父路由结束。然后onEnter hook会从最外层的父路由开始直到最下层子路由结束。

继续我们上面的例子，如果一个用户点击链接，从 `/messages/5` 跳转到 `/about`，下面是这些 hook 的执行顺序：
1. `/messages/:id` 的 `onLeave`
2. `/inbox` 的 `onLeave`
3. `/about` 的 `onEnter`

### 替换的配置方式

因为 route 一般被嵌套使用，所以使用 JSX 这种天然具有简洁嵌套型语法的结构来描述它们的关系非常方便。然而，如果你不想使用 JSX，也可以直接使用原生 route 数组对象。

上面我们讨论的路由配置可以被写成下面这个样子：

```jsx
const routeConfig = [
  { path: '/',
    component: App,
    indexRoute: { component: Dashboard },
    childRoutes: [
      { path: 'about', component: About },
      { path: 'inbox',
        component: Inbox,
        childRoutes: [
          { path: '/messages/:id', component: Message },
          { path: 'messages/:id',
            onEnter: function (nextState, replaceState) {
              replaceState(null, '/messages/' + nextState.params.id)
            }
          }
        ]
      }
    ]
  }
]

React.render(<Router routes={routeConfig} />, document.body)
```

## 路由匹配原理

路由拥有三个属性来决定是否“匹配“一个 URL：
1. 嵌套关系
2. 路径语法
3. 优先级

### 嵌套关系

React Router 使用路由嵌套的概念来让你定义 view 的嵌套集合，当一个给定的 URL 被调用时，整个集合中（命中的部分）都会被渲染。嵌套路由被描述成一种树形结构。React Router 会深度优先遍历整个路由配置来寻找一个与给定的 URL 相匹配的路由。

### 路径语法

路由路径是匹配一个（或一部分）URL 的 一个字符串模式。大部分的路由路径都可以直接按照字面量理解，除了以下几个特殊的符号：
* `:paramName` – 匹配一段位于 `/`、`?` 或 `#` 之后的 URL。 命中的部分将被作为一个参数
* `()` – 在它内部的内容被认为是可选的
* `*` – 匹配任意字符（非贪婪的）直到命中下一个字符或者整个 URL 的末尾，并创建一个 splat 参数
```jsx
<Route path="/hello/:name">         // 匹配 /hello/michael 和 /hello/ryan
<Route path="/hello(/:name)">       // 匹配 /hello, /hello/michael 和 /hello/ryan
<Route path="/files/*.*">           // 匹配 /files/hello.jpg 和 /files/path/to/hello.jpg
```
如果一个路由使用了相对路径，那么完整的路径将由它的所有祖先节点的路径和自身指定的相对路径拼接而成。使用绝对路径可以使路由匹配行为忽略嵌套关系。

### 优先级

最后，路由算法会根据定义的顺序自顶向下匹配路由。因此，当你拥有两个兄弟路由节点配置时，你必须确认前一个路由不会匹配后一个路由中的路径。例如，千万不要这么做：

```jsx
<Route path="/comments" ... />
<Redirect from="/comments" ... />
```

## History

React Router 是建立在 `history` 之上的。 简而言之，一个 history 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 location 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。
常用的 history 有三种形式， 但是你也可以使用 React Router 实现自定义的 history。

* browserHistory
* hashHistory
* createMemoryHistory

你可以从 React Router 中引入它们：

```jsx
// JavaScript 模块导入（译者注：ES6 形式）
import { browserHistory } from 'react-router'
```

然后将它们传递给`<Router>`:

```jsx
render(
  <Router history={browserHistory} routes={routes} />,
  document.getElementById('app')
)
```

### browserHistory

Browser history 是使用 React Router 的应用推荐的 history。它使用浏览器中的 [History API](https://developer.mozilla.org/en-US/docs/Web/API/History) 用于处理 URL，创建一个像example.com/some/path这样真实的 URL 。

#### 服务器配置

服务器需要做好处理 URL 的准备。处理应用启动最初的 `/` 这样的请求应该没问题，但当用户来回跳转并在 `/accounts/123` 刷新时，服务器就会收到来自 `/accounts/123` 的请求，这时你需要处理这个 URL 并在响应中包含 JavaScript 应用代码。
一个 express 的应用可能看起来像这样的：
```nodejs
const express = require('express')
const path = require('path')
const port = process.env.PORT || 8080
const app = express()

// 通常用于加载静态资源
app.use(express.static(__dirname + '/public'))

// 在你应用 JavaScript 文件中包含了一个 script 标签
// 的 index.html 中处理任何一个 route
app.get('*', function (request, response){
  response.sendFile(path.resolve(__dirname, 'public', 'index.html'))
})

app.listen(port)
console.log("server started on port " + port)
```

#### IE8, IE9 支持情况

如果我们能使用浏览器自带的 window.history API，那么我们的特性就可以被浏览器所检测到。如果不能，那么任何调用跳转的应用就会导致 全页面刷新，它允许在构建应用和更新浏览器时会有一个更好的用户体验，但仍然支持的是旧版的。

你可能会想为什么我们不后退到 hash history，问题是这些 URL 是不确定的。如果一个访客在 hash history 和 browser history 上共享一个 URL，然后他们也共享同一个后退功能，最后我们会以产生笛卡尔积数量级的、无限多的 URL 而崩溃。

### hashHistory

Hash history 使用 URL 中的 hash（#）部分去创建形如 `example.com/#/some/path` 的路由。

#### 我应该使用 `createHashHistory`吗？

Hash history 不需要服务器任何配置就可以运行，如果你刚刚入门，那就使用它吧。但是我们不推荐在实际线上环境中用到它，因为每一个 web 应用都应该渴望使用 `browserHistory`。

#### 像这样 `?_k=ckuvup` 没用的在 URL 中是什么？

当一个 history 通过应用程序的 push 或 replace 跳转时，它可以在新的 location 中存储 “location state” 而不显示在 URL 中，这就像是在一个 HTML 中 post 的表单数据。

在 DOM API 中，这些 hash history 通过 window.location.hash = newHash 很简单地被用于跳转，且不用存储它们的location state。但我们想全部的 history 都能够使用location state，因此我们要为每一个 location 创建一个唯一的 key，并把它们的状态存储在 session storage 中。当访客点击“后退”和“前进”时，我们就会有一个机制去恢复这些 location state。

### createMemoryHistory

Memory history 不会在地址栏被操作或读取。这就解释了我们是如何实现服务器渲染的。同时它也非常适合测试和其他的渲染环境（像 React Native ）。

和另外两种history的一点不同是你必须创建它，这种方式便于测试。

```jsx
const history = createMemoryHistory(location)
```

### 实现示例

```jsx
import React from 'react'
import { render } from 'react-dom'
import { browserHistory, Router, Route, IndexRoute } from 'react-router'

import App from '../components/App'
import Home from '../components/Home'
import About from '../components/About'
import Features from '../components/Features'

render(
  <Router history={browserHistory}>
    <Route path='/' component={App}>
      <IndexRoute component={Home} />
      <Route path='about' component={About} />
      <Route path='features' component={Features} />
    </Route>
  </Router>,
  document.getElementById('app')
)
```

## 动态路由

React Router 适用于小型网站，对于大型应用来说，一个首当其冲的问题就是所需加载的 JavaScript 的大小。程序应当只加载当前渲染页所需的 JavaScript。有些开发者将这种方式称之为“代码分拆” —— 将所有的代码分拆成多个小包，在用户浏览过程中按需加载。

对于底层细节的修改不应该需要它上面每一层级都进行修改。举个例子，为一个照片浏览页添加一个路径不应该影响到首页加载的 JavaScript 的大小。也不能因为多个团队共用一个大型的路由配置文件而造成合并时的冲突。

路由是个非常适于做代码分拆的地方：它的责任就是配置好每个 view。

React Router 里的路径匹配以及组件加载都是异步完成的，不仅允许你延迟加载组件，并且可以延迟加载路由配置。在首次加载包中你只需要有一个路径定义，路由会自动解析剩下的路径。

Route 可以定义 `getChildRoutes`，`getIndexRoute` 和 `getComponents` 这几个函数。它们都是异步执行，并且只有在需要时才被调用。我们将这种方式称之为 “逐渐匹配”。 React Router 会逐渐的匹配 URL 并只加载该 URL 对应页面所需的路径配置和组件。

如果配合 `webpack` 这类的代码分拆工具使用的话，一个原本繁琐的构架就会变得更简洁明了。

```jsx
const CourseRoute = {
  path: 'course/:courseId',

  getChildRoutes(location, callback) {
    require.ensure([], function (require) {
      callback(null, [
        require('./routes/Announcements'),
        require('./routes/Assignments'),
        require('./routes/Grades'),
      ])
    })
  },

  getIndexRoute(location, callback) {
    require.ensure([], function (require) {
      callback(null, require('./components/Index'))
    })
  },

  getComponents(location, callback) {
    require.ensure([], function (require) {
      callback(null, require('./components/Course'))
    })
  }
}
```

## 跳转前确认

React Router 提供一个 `routerWillLeave` 生命周期钩子，这使得 React 组件可以拦截正在发生的跳转，或在离开 route 前提示用户。`routerWillLeave` 返回值有以下两种：

1. `return false` 取消此次跳转
2. `return` 返回提示信息，在离开 route 前提示用户进行确认。

## 服务端渲染

服务端渲染与客户端渲染有些许不同，因为你需要：
1. 发生错误时发送一个 `500` 的响应
2. 需要重定向时发送一个 30x 的响应
在渲染之前获得数据 (用 router 帮你完成这点)
为了迎合这一需求，你要在 [Router API](https://github.com/ReactTraining/react-router/blob/v3/docs/API.md) 下一层使用：
* 使用 `match` 在渲染之前根据 `location` 匹配 route
* 使用 `RoutingContext` 同步渲染 route 组件
它看起来像一个虚拟的 JavaScript 服务器：

```jsx
import { renderToString } from 'react-dom/server'
import { match, RoutingContext } from 'react-router'
import routes from './routes'

serve((req, res) => {
  // 注意！这里的 req.url 应该是从初始请求中获得的
  // 完整的 URL 路径，包括查询字符串。
  match({ routes, location: req.url }, (error, redirectLocation, renderProps) => {
    if (error) {
      res.send(500, error.message)
    } else if (redirectLocation) {
      res.redirect(302, redirectLocation.pathname + redirectLocation.search)
    } else if (renderProps) {
      res.send(200, renderToString(<RoutingContext {...renderProps} />))
    } else {
      res.send(404, 'Not found')
    }
  })
})
```

至于加载数据，你可以用 `renderProps` 去构建任何你想要的形式——例如在 route 组件中添加一个静态的 `load` 方法，或如在 route 中添加数据加载的方法——由你决定。
