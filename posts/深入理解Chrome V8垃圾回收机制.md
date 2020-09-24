
## JavaScript的内存管理

与其他需要手动管理内存的语言不通，在JavaScript中，当我们创建变量（对象，字符串等）的时候，系统会自动给对象分配对应的内存。

```js
var n = 123; // 给数值变量分配内存
var s = "azerty"; // 给字符串分配内存

var o = {
  a: 1,
  b: null
}; // 给对象及其包含的值分配内存

// 给数组及其包含的值分配内存（就像对象一样）
var a = [1, null, "abra"]; 

function f(a){
  return a + 2;
} // 给函数（可调用的对象）分配内存

// 函数表达式也能分配一个对象
someElement.addEventListener('click', function(){
  someElement.style.backgroundColor = 'blue';
}, false);
```

当系统发现这些变量不再被使用的时候，会自动释放（垃圾回收）这些变量的内存，开发者不用过多的关心内存问题。

虽然这样，我们开发过程中也需要了解JavaScript的内存管理机制，这样才能避免一些不必要的问题，比如下面代码：

```js
{}=={} // false
[]==[] // false
''=='' // true
```

在JavaScript中，数据类型分为两类，简单类型和引用类型，对于简单类型，内存是保存在栈（stack）空间中，复杂数据类型，内存是保存在堆（heap）空间中。

* 基本类型：这些类型在内存中分别占有固定大小的空间，他们的值保存在栈空间，我们通过按值来访问的
* 引用类型：引用类型，值大小不固定，栈内存中存放地址指向堆内存中的对象。是按引用访问的。

而对于栈的内存空间，只保存简单数据类型的内存，由操作系统自动分配和自动释放。而堆空间中的内存，由于大小不固定，系统无法无法进行自动释放，这个时候就需要JS引擎来手动的释放这些内存。

## 为什么需要垃圾回收机制

在Chrome中，v8被限制了内存的使用，（64位约1.4G/1464MB ， 32位约0.7G/732MB），为什么要限制呢？

1. 表层原因是，V8最初为浏览器而设计，不太可能遇到用大量内存的场景，
2. 深层原因是，V8的垃圾回收机制的限制，（如果清理大量的内存垃圾是很耗时间，这样回引起JavaScript线程暂停执行的时间，那么性能和应用直线下降）

前面我们说到，栈内的内存，操作系统会自动进行内存分配和内存释放，而堆中的内存，有JS引擎（如Chrome的V8）手动进行释放，当我们代码的按照正确的写法时，会使得JS引擎的垃圾回收机制无法正确的对内存进行释放（内存泄露），从而使得浏览器占用的内存不断增加，进而导致JavaScript和应用、操作系统性能下降。

## Chrome 垃圾回收算法

在JavaScript中，其实绝大多数的对象存活周期都很短，大部分在经过一次的垃圾回收之后，内存就会被释放掉，而少部分的对象存活周期将会很长，一直是活跃的对象，不需要被回收。为了提高回收效率，V8 将堆分为两类新生代和老生代，新生代中存放的是生存时间短的对象，老生代中存放的生存时间久的对象。

新生区通常只支持 1～8M 的容量，而老生区支持的容量就大很多了。对于这两块区域，V8 分别使用两个不同的垃圾回收器，以便更高效地实施垃圾回收。

* 副垃圾回收器 - Scavenge：主要负责新生代的垃圾回收。
* 主垃圾回收器 - Mark-Sweep & Mark-Compact：主要负责老生代的垃圾回收。

### 新生代垃圾回收器 - Scavenge

在JavaScript中，任何对象的声明分配到的内存，将会先被放置在新生代中，而因为大部分对象在内存中存活的周期很短，所以需要一个效率非常高的算法。在新生代中，主要使用`Scavenge`算法进行垃圾回收，`Scavenge`算法是一个典型的牺牲空间换取时间的复制算法，在占用空间不大的场景上非常适用。





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