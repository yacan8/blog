---
title: Promise实现原理
date: 2018-02-26 17:25:09
tags:
- promise
- JavaScript
---

## 前言

在Promise没有出现之前，异步编程需要通过回调的方式进行完成，当回调函数嵌套过多时，会使代码丑化，也降低了代码的可理解性，后期维护起来会相对困难，Promise是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了用法，原生提供了Promise对象，本文主要针对Promise/A+规范，实现一个小型的Promise对象。

## Promise/A+ 规范

Promise规范有很多，如Promise/A，Promise/B，Promise/D 以及 Promise/A 的升级版Promise/A+，因为ES6主要用的是Promise/A+规范，该规范内容也比较多，我们挑几个简单的说明下:

1. Promise本身是一个状态机，每一个Promise实例只能有三个状态，`pending`、`fulfilled`、`reject`，状态之间的转化只能是`pending->fulfilled`、`pending->reject`，状态变化不可逆。
2. Promise有一个`then`方法，该方法可以被调用多次，并且返回一个Promise对象（返回新的Promise还是老的Promise对象，规范没有提）。
3. 支持链式调用。
4. 内部保存有一个value值，用来保存上次执行的结果值，如果报错，则保存的是异常信息。

具体规范可参考:
* 英文版：[promisesaplus.com/](https://promisesaplus.com/)
* 中文版：[【翻译】Promises/A+规范](http://www.ituring.com.cn/article/66566)

## 实现

由于Promise为状态机，我们需先定义状态

```js
var PENDING = 0; // 进行中
var FULFILLED = 1; // 成功
var REJECTED = 2; // 失败
```

### 基本代码

```js
function Promise(fn) {
  var state = PENDING;  // 存储PENDING, FULFILLED或者REJECTED的状态
  var value = null;  // 存储成功或失败的结果值
  var handlers = []; // 存储成功或失败的处理程序，通过调用`.then`或者`.done`方法

  // 成功状态变化
  function fulfill(result) {
      state = FULFILLED;
      value = result;
      handlers.forEach(handle); // 处理函数，下文会提到
      handlers = null;
   }

  // 失败状态变化
  function reject(error) {
      state = REJECTED;
      value = error;
      handlers.forEach(handle); // 处理函数，下文会提到
      handlers = null;
  }
}
```

### 实现resolve方法

resolve方法可以接受两种参数，一种为普通的值/对象，另外一种为一个Promise对象，如果是普通的值/对象，则直接把结果传递到下一个对象；

如果是一个 Promise 对象，则必须先等待这个子任务序列完成。

```js
function Promise(fn) {
  ...
  function resolve(result) {
      try {
        var then = getThen(result);
        if (then) {
          doResolve(then.bind(result), resolve, reject)
          return;
        }
        fulfill(result);
      } catch (e) {
        reject(e);
      }
  }
  ...
}
```
resolve需要两个辅助方法`getThen`、和`doResolve`。
```js
// getThen 检查如果value是一个Promise对象，则返回then方法等待执行完成。
function getThen(value) {
  var t = typeof value;
  if (value && (t === 'object' || t === 'function')) {
    var then = value.then;
    if (typeof then === 'function') {
      return then;
    }
  }
  return null;
}
// 异常参数检查函数，确保onFulfilled和onRejected两个函数中只执行一个且只执行一次，但是不保证异步。
function doResolve(fn, onFulfilled, onRejected) {
  var done = false;
  try {
    fn(
      function(value) {
        if (done) return;
        done = true;
        onFulfilled(value);
      },
      function(reason) {
        if (done) return;
        done = true;
        onRejected(reason);
      }
    );
  } catch(ex) {
    if (done) return;
    done = true;
    onRejected(ex);
  }
}
```

上面已经完成了一个完整的内部状态机，但我们并没有暴露一个方法去解析或则观察 Promise 。现在让我们开始解析 Promise ：

```js
function Promise(fn) {
  ...
  doResolve(fn, resolve, reject);
}
```

如你所见，我们复用了doResolve，因为对于初始化的fn也要对其进行控制。fn允许调用resolve或则reject多次，甚至抛出异常。这完全取决于我们去保证promise对象仅被resolved或则rejected一次，且状态不能随意改变。

### then方法实现

在实现`then`方法之前，我们这里实现了一个执行方法done，该方法用来处理执行then方法的回调函数，一下为promise.done(onFullfilled, onRejected)方法的几个点。

* onFulfilled 和 onRejected 两者只能有一个被执行，且执行次数为一
* 该方法仅能被调用一次, 一旦调用了该方法，则 promise 链式调用结束
* 无论是否 promise 已经被解析，都可以调用该方法

```js
function Promise(fn) {
  ...
  // 不同状态，进行不同的处理
  function handle(handler) {
    if (state === PENDING) {
      handlers.push(handler);
    } else {
      if (state === FULFILLED && typeof handler.onFulfilled === 'function') {
        handler.onFulfilled(value);
      }
      if (state === REJECTED && typeof handler.onRejected === 'function') {
        handler.onRejected(value);
      }
    }
  }

  this.done = function (onFulfilled, onRejected) {
    // 保证异步
    setTimeout(function () {
      handle({onFulfilled: onFulfilled, onRejected: onRejected});
    }, 0);
  }
}
```

当 Promise 被 resolved 或者 rejected 时，我们保证 handlers 将被通知。

then方法
```js
function Promise(fn) {
  ...
  this.then = function(onFulfilled, onRejected) {
    var self = this;
    return new Promise(function (resolve, reject) {
      self.done(function (result) {
        if (typeof onFulfilled === 'function') {
          try {
            // onFulfilled方法要有返回值！
            return resolve(onFulfilled(result));
          } catch (ex) {
            return reject(ex);
          }
        } else {
          return resolve(result);
        }
      }, function (error) {
        if (typeof onRejected === 'function') {
          try {
            return resolve(onRejected(error));
          } catch (ex) {
            return reject(ex);
          }
        } else {
          return reject(error);
        }
      });
    });
  }
}
```
catch方法，我们直接调用then处理异常

```js
this.catch = function(errorHandle) {
  return this.then(null, errorHandle);
}
```

以上为promise实现原理~
