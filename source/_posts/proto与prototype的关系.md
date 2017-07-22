---
title: __proto__ 与 prototype 的关系
tags:
  - 前端
  - JavaScript
date: 2017-01-22 09:21:37
---

之前常听说 JavaScript 的面向对象是构建于原型链（protype chain）上的，通过逐级向上查找直至 Object 的原型为止，但是具体的工作机制不太清楚，今天查阅了资料，才发现原来自己之前把 `__proto__` 与 `prototype` 混为一谈了（冏o(╯□╰)o）。

<!--more-->

## 结论

先抛出来结论：

1.  `__proto__` 是原型链查询中实际用到的，它总是指向 `prototype`；
2.  `prototype` 在定义构造函数时自动创建，它总是被 `__proto__` 所指。

从上面两点我们还可以推出 `prototype` 只能作为构造函数的属性，而 `__proto__` 可以作为任意对象的属性。

## 细探

接下来通过一段代码具体的研究一下：

```javascript
function Foo(y) { this.y = y; };
Foo.prototype.x = 10;
FOo.prototype.calculate = function () { … };
let b = new Foo(20);
let c = new Foo(30);
```

这段代码构建的原型链可以用下面的图来表示：![原型链示意图](https://ww3.sinaimg.cn/large/006tKfTcgy1fbyc6irh2aj30hf0aw3z1.jpg)

看上去是不是有些晕？ 没关系，看完这篇文章就会觉得很简单了。

### 原型链解释

首先聚焦于图的中央，也就是蓝色和灰色的块。代码的第一行 `function Foo(age) { ... };` 定义了构造函数 Foo，此时 Foo 的原型便自动创建，它的 constructor 属性指向 Foo，Foo 的 prototype 属性指向刚刚创建的原型：

```javascript
Foo.prototype.constructor === Foo; // true
```

接着再看图的左部，也就是黄绿色的块。代码的第四、五行通过 `let b = new Foo(20); let c = new Foo(20);` 创建了对象 b 和 c，每当我们通过 `new Foo(x)` 创建对象时，JavaScript 内部会首先创建一个新的对象，将其  `__proto__` 属性指向 Foo 的原型，也就是 Foo.prototype，然后将构造函数中的 `this` 指向刚刚创建的新对象，最后再执行构造函数中的代码：

```javascript
b.proto === Foo.prototype; // true
c.proto === Foo.prototype; // true
```

最后来说说剩下的紫色的块。

既然原型链是一条链，那么它就一定会有起始点。文章的开头就说过原型链查找是逐级向上查找直至 Object 的原型为止，所以说起始点就是 `Object.prototype`，自然 `Object.prototype.__proto__` 的值为 null（因为已经到了链头，链结束了）。

尽管 Foo 是构造函数，但它仍逃不出函数的范畴，所以它的 `__proto__` 指向函数的原型，也就是 `Function.prototype`。

一切原型链的终点都是 `Object.prototype`，所以 `Foo.prototype.__proto__` 和 `Function.prototype.__proto__` 均指向它。

```javascript
Foo.proto === Function.prototype; // true
Foo.prototype.proto === Object.prototype; //true
Function.prototype.proto === Object.prototype; // true
Object.prototype.proto === null; // true
```

### 原型链工作原理

一句话解释就是：原型链查找就是通过\__ proto\__ 查找，直至 Object.prototype时结束，通过几个具体的例子来说明一下：

1.  `b.y` 在 b 中找到 y，结果为 20；
2.  `b.x` 在 b 中未找到 x，接着通过 b\.__proto\__ 在 Foo.prototype 中找到 x，结果为 10；
3.  `b.toString()` 在 b 中未找到 toString 方法；接着通过 b.\__proto\__ 在 Foo.prototype 中寻找，未找到；继续通过 Foo.prototype.\__proto\__ 在 Object.prototype 中寻找，找到，结果为 `[object Object]`。
4.  `b.noFunc()` 依次在 b、Foo.prototype、Object.prototype 中寻找，均未找到，抛出错误。

## 参考链接

1.  [StackOverflow - \__proto\__ vs prototype](http://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript)。