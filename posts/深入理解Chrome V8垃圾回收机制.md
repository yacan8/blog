
## JavaScript的内存管理

## 为什么需要垃圾回收机制

存在限制：Chrome限制了v8内存的使用，（64位约1.4G/1464MB ， 32位约0.7G/732MB），这将导致无法直接操作大内存对象。
限制原因：①表层原因为V8最初为浏览器而设计，不太可能遇到用大量内存的场景，②深层原因是V8的垃圾回收机制的限制，（如果清理大量的内存垃圾是很耗时间，这样回引起JavaScript线程暂停执行的时间，那么性能和应用直线下降）
限制选项：V8的内存也是可以修改的，主要考虑使用场景，修改的方法

## Chrome 分代回收

为什么需要分代回收

### 新生代垃圾回收 - Scavenge

定义、描述

有点、缺点

晋升

### 老生代垃圾回收 - Mark-Sweep & Mark-Compact

#### mark-sweep

#### mark-compact

## 全停顿 Stop-The-World

## Orinoco

### 增量标记 - Incremental marking

### 懒清理 - Lazy sweeping

2011年，V8从全停顿切换至增量标记。
2018年，Chrome64和Node.js V10启动并发标记（Concurrent），同时添加并行（Parallel）技术，使得垃圾回收时间大幅度缩短。

### 并发 - Concurrent

### 并行 - Parallel

### 空闲时垃圾回收器

## 参考文献

* [NodeJs internals: V8 & garbage collector](https://medium.com/voodoo-engineering/nodejs-internals-v8-garbage-collector-a6eca82540ec)

* [「译」Orinoco: V8的垃圾回收器](https://zhuanlan.zhihu.com/p/55917130)

* [Chrome 浏览器垃圾回收机制与内存泄漏分析](https://www.cnblogs.com/LuckyWinty/p/11739573.html)

* [JavaScript中的堆栈以及垃圾回收机制](https://blog.csdn.net/sxs7970/article/details/97569470)

* [超详细的node/v8/js垃圾回收机制](https://www.cnblogs.com/ysk123/p/11656452.html)

* [垃圾回收技术](https://www.cnblogs.com/suihang/p/12840671.html)

* [4类 JavaScript 内存泄漏及如何避免](https://jinlong.github.io/2016/05/01/4-Types-of-Memory-Leaks-in-JavaScript-and-How-to-Get-Rid-Of-Them/)

* [内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)