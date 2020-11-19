



## Nodejs


```ruby
   ┌───────────────────────┐
┌─>│        timers         │<————— 执行 setTimeout()、setInterval() 的回调
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
│  │     pending callbacks │<————— 执行由上一个 Tick 延迟下来的 I/O 回调（待完善，可忽略）
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
│  │     idle, prepare     │<————— 内部调用（可忽略）
│  └──────────┬────────────┘     
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
|             |                   ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │ - (执行几乎所有的回调，除了 close callbacks 以及 timers 调度的回调和 setImmediate() 调度的回调，在恰当的时机将会阻塞在此阶段)
│  │         poll          │<─────┤  connections, │ 
│  └──────────┬────────────┘      │   data, etc.  │ 
│             |                   |               | 
|             |                   └───────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
|  ┌──────────┴────────────┐      
│  │        check          │<————— setImmediate() 的回调将会在这个阶段执行
│  └──────────┬────────────┘
|             |<-- 执行所有 Next Tick Queue 以及 MicroTask Queue 的回调
│  ┌──────────┴────────────┐
└──┤    close callbacks    │<————— socket.on('close', ...)
   └───────────────────────┘
```

- 定时器检测阶段（timers）：本阶段执行 timers 的回调，即 setTimeout、setInterval 里面的回调函数
- I / O 事件回调阶段（I / O callbacks）：执行延迟到下一个循环迭代的 I / O 回调，即上一轮循环中未被执行的一些 I / O 回调
- 闲置阶段（idle，prepare）：仅供内部使用
- 轮询阶段（poll）：检索新的 I / O 事件；执行与 I / O 相关的回调（几乎所有情况下，除了关闭的回调函数，那些计时器和 setImmediate 调度之外），其余情况 node 将在适当的时候在此阻塞
- 检查阶段（check）：setImmediate 回调函数将在此阶段执行
- 关闭事件回调阶段（close callback）：一些关闭的回调函数，如 `socket.on('close', ...)`

### poll

![poll](https://camo.githubusercontent.com/1633630fb3740e71a86330df5c0e9a44e5c6127c/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323032302f332f322f313730393935316536356666653030653f696d616765736c696d)



这里主要说明 node11 前后的差异，因为 node11 之后一些特性向浏览器看齐，总的变化一句话来说就是

如果是 node11 版本一旦执行一个阶段里的一个宏任务（setTimeout、setInterval、setImmediate）就会立刻执行对应的微任务队列以及微任务产生的微任务



## 浏览器

```ruby
   ┌───────────────────────┐
┌─>│        timers         │<————— 执行一个 MacroTask Queue 的回调
│  └──────────┬────────────┘
|             |<-- 执行所有 MicroTask Queue 的回调
| ────────────┘
```

async/await

新版本chrome以后，提升await优先级，await以后立刻向队列中推进微任务



## node 和 浏览器事件循环的主要区别

两者主要的区别在于浏览器中的微任务是在每个相应的宏任务中执行的，而 nodejs 中的微任务则是在不同阶段之间执行的。 