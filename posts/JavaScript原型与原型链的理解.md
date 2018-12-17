---
title: JavaScript原型与原型链的理解
date: 2018-05-13 20:02:03
tags:
- JavaScript
---

### 引言
原型与原型链一直是JavaScript中的重点与难点，虽然老生常谈了，但是还是需要对其进行深入与巩固。。。。

### 正文
首先是原型，要理解原型，需要知道三个东西;
`prototype`：每个`函数`都会有的属性，它指向函数的原型对象。（注意：函数对象才有，普通对象没有。）
`__proto__`：每一个对象都会有个属性，当然，函数也属于对象，所以函数也有这个属性，它指向构造函数的原型对象;
`constructor`：原型对象上的一个属性，指向对象的构造函数;

下面用示例来验证一下其关系;
```js
function Pig(name) {
  this.name = name;
}
var peppa = new Pig('peppa');
```
用`new`关键字实例化一个Pig对象，返回一个普通对象pie示例，在实例化的时候，Pig的prototype上的属性会作为原型对象赋值给实例。也就是说peppa的原型，是从Pig的prototype引用来的，即`peppa.__proto__ === Pig.prototype`;
```js
peppa.__proto__ === Pig.prototype;
// true;
```
Pig是一个函数对象，它是Function的一个实例，所以`Pig.__proto__ === Function.prototype`一定为true;
```js
Pig.__proto__ === Function.prototype;
// true
```
上面说到的constructor，它指向原型对象构造函数属性，也就是`peppa.__proto__.constructor == Pig`;
```js
pie.__proto__.constructor == Pig;
// true
```

当然我们有`peppa.__proto__ === Pig.prototype`，所以`Pig.prototype.constructor == Pig`也为true;
```js
Pig.prototype.constructor == Pig
```

下面我们来说一下Pig、Function、Object之间的关系；
首先，Pig为Function的实例，所以有`Pig.__proto__ === Function.prototype`为true；
```js
Pig.__proto__ === Function.prototype;
// true
```

然后由于构造函数创建的时候会给其函数加上prototype属性，方面后面实例的引用，prototype属于普通对象，为Object的实例，有：
```js
Pig.prototype.__proto__ === Object.prototype;
// true
```

由上可以知道，所有构造函数的原型对象的`__proto__`都指向Object.prototype：
```js
Function.prototype.__proto__ === Object.prototype
// true
```

另外，Object为一个对象，可以认为为某一个Function的实例返回，所以有：
```js
Object.__proto__ === Function.prototype
// true
```



至此，得到链条`pie.__proto__ === Pig.prototype` => `Pig.__proto__ === Function.prototype` => `Function.prototype.__proto__ === Object.prototype`;
那么`Object.prototype.__proto__`又指向谁，JS世界里万物皆对象，Object似乎已经到了原型链的顶端，果然不出我所料，它确实是null；
```js
Object.prototype.__proto__ === null;
// true
```

接着说一下原型链。正如你在上面图中所看到的，JS在创建对象的时候，会在新对象上产生一个__proto__的属性，这个属性指向了它构造函数的原型的prototype。由此一级一级向上直到到达Object.prototype.proto === null的这个链条我们称之为原型链。

### 结束
关于原型与原型链大概理解如上，大部分继承都是基于原型与原型链完成的。。。
