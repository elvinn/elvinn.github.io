---
title: 给 Number 打 Call
date: 2017-12-24 09:48:46
tags:
	- 前端
	- JavaScript
---

最近“打 call”这个词突然流行起来，我想到 `call` 在 JavaScript 中可是个重要的方法，那么能不能用 JavaScript 也来玩一把打 call 呢？于是我在 Number 上实验了下，写出来的代码让我自己都一脸懵逼了，大家能猜到下面这段代码的输出结果吗？

```javascript
Number.call(Number, 1, 2, 3);
Number.call.call(Number, 1, 2, 3);
Number.call.call.call(Number, 1, 2, 3);
Number.call === Number.call.call;
Number.call === Number.call.call.call;
```

<!--more-->

## 结果初探

![打 call 的结果](https://ww2.sinaimg.cn/large/0060lm7Tly1fn3lud9mgyj30i80aegnn.jpg)

 不知道大家有没有猜中运行结果呢？反正我看到这个结果的时候是崩溃的（内心活动：为什么相等的三个函数，对相同的参数，返回的结果却不一样？？不愧是 JavaScript！）不过虽然结果看起来违反直觉，但是从上述代码中还是可以得出两个好消息的：

1. `Number.call(Number, 1, 2, 3)` 这段代码的运行结果是符合预期的，它其实就是将 Number 中的 this 指向 Number，然后执行 Number(1, 2, 3)，和直接执行 `Number(1, 2, 3)` 作用相同，效果就是对传入的第一个参数尝试转化为数字并返回，我们可以通过下列代码验证一下：

   ```javascript
   Number.call(Number, 1, 2, 3); // output is 1
   Number(1, 2, 3); // output is 1

   Number.call(Number, '1', '2', '3'); // output is 1
   Number('1', '2', '3'); // output is  1

   Number.call(Number, 'do', 'call'); // output is NaN(Not a Number)
   Number('do', 'call'); // output is NaN(Not a Number)
   ```

2. `Number.call`、`Number.call.call` 和 `Number.call.call.call` 用全等符号的判断结果是 `true`，这说明它们是同一个函数， 那么它们到底是指向哪一个函数呢？仔细想一下，`Number` 是函数，`Number.call` 是函数，`Number.call.call` 也是函数，那么访问一个函数的 `call` 方法会不会就是访问 Function 原型上的 call 方法呢？同样地，我们通过下列代码验证一下：

   ```javascript
   typeof Number;  // output is "function"
   typeof Number.call; // output is "function"
   typeof Number.call.call; // output is "function"

   Number.call === Function.prototype.call; // output is true
   Number.call.call === Function.prototype.call; // output is true
   Number.call.call.call === Function.prototype.call; // output is true
   ```

> 关于原型链更深入的知识，可以参考我之前写的博客 - [\_\_proto\_\_ 与 prototype 的关系](http://segmentfault.me/2017/01/22/proto%E4%B8%8Eprototype%E7%9A%84%E5%85%B3%E7%B3%BB/)

虽然有了一些收获，但是为什么对于相同的函数 `Function.prototype.call`，输入相同的参数 `(Number, 1, 2, 3)`，有时候得到的结果是 1，有时候得到结果却是 2 呢？

## 深入探究

既然已经定位到了 `Function.prototype.call` 这里，那么我们可以自己写一个 `myCall` 函数来代替 `call` 函数，从而进一步地探究这个函数内部发生了什么。不要被自己写一个 `call` 函数吓到，毕竟它的作用很简单：将调用者函数中的 this 改为传入的第一个参数，然后再依次传入后面的参数并执行函数。所以我们可以先编写一个这样的函数：

```javascript
Function.prototype.myCall = function myCall(obj, ...args) {
  console.log(`this is ${this.name}, obj is ${obj}, arguments are ${args}`);
  console.log('------------------');
  const newFunc = this.bind(obj);
  return newFunc(...args);
}
```

在上面的代码中，关键在于 `const newFunc = this.bind(obj);`，这里的 `this` 指的就是调用者函数，然后通过 `this.bind(obj)` 将调用者函数中的 this（注意：这个 this 是调用者函数中的 this，而不是 myCall 中的 this） 变为了 obj。

> 上面只是为了探究问题而实现的简单版 call 函数，实际上 call 函数还会对传入的第一个参数做校验和替换，例如传入基本数据类型（primitive values）时会被转化为对象，不过这些与本次问题无关，可以不用关注 ^_^

然后让我们来运行一下 myCall，看看结果如何：

![myCall 执行结果](https://ww1.sinaimg.cn/large/0060lm7Tly1fn3mng6p9ej30zk0gqgs5.jpg)

运行的结果和原生的 call 函数相同，说明这个简单版的 myCall 函数实现了目标。在上图中，绿块和蓝块的输出相同，可以把它们俩儿归为一类，所以接下来主要对红块和绿块中的结果进行分析。

### Number.myCall

从红块中，我们可以看出函数调用只发生了一次，且 this 为 Number，obj 为 Number，arguments 为 1、2 和 3，所以 `Number.myCall(Number, 1, 2, 3)` 其实相当于调用了 `Number.bind(Number)(1, 2, 3)`，所以最后结果为 1，这与我们在第一节得出的结论相同。

### Number.myCall.myCall

从绿块中，我们可以看出函数调用发生了两次：

1. 对于第一次调用，将输出的 this、obj 和 arguments 带入到 myCall 中，可以发现其实是执行了 `myCall.bind(Number)(1, 2, 3)`；
2. 对于第二次调用，同样将输出的 this、obj 和 arguments 带入到 myCall 中，可以发现其实是执行了 `Number.bind(1)(2, 3)`，所以结果与执行 `Number(2, 3)`相同，也就是 2。

———————————— 前方高能预警！！会比较绕！！请准备！请准备！————————————

对比 `Number.myCall` 和 `Number.myCall.myCall` 的第一次调用，可以发现其实 obj 和 arguments 都没有变化，**只是 this 的指向发生了变化**：对于 `Number.myCall` 而言，myCall 的调用者是 `Number`，所以 this 指向了 Number；对于 `Number.myCall.myCall` 而言，  最后一个 myCall 的调用者是 `Number.myCall`，所以 this 指向了 myCall。再进一步，我们看看 `Number.myCall.myCall.myCall`，最后一个 myCall 的调用者是 `Number.myCall.myCall`，也就是 Funct.prototype.myCall，所以 this 指向了 myCall，和 Number.myCall.myCall 调用时 this 指向相同，所以输出的结果也相同，都是 2。

———————————————————— 预警结束~~~ ————————————————————

所以说呀，尽管 Number.myCall 和 Number.myCall.myCall 是同一个函数，而且传入的参数也相同，但是由于函数内部的 this 指向不同，导致了函数最后的执行结果不同。这就是基于 “原型链” 的动态语言 JavaScript 的“神奇”之处，在基于类的语言如 C++、Java 中可是难以碰到这等好玩的事情呢~



## 写在最后

如果读者坚持看到了这里而且还没有晕的话，我真心地佩服你 👍👍 想当初我看到这个执行结果，到最后弄明白的一个多小时里，我是一直处于懵逼的状态中。这里特别感谢和我一起探究的 [avwo（他出品的代理工具 whistle 广受好评，大家可以试试~）](https://github.com/avwo/) 和提供文章题目的 [litten](http://litten.me/)。

最后为了避免大家觉得我是疯子居然写出这种代码来，我在这里说明一下，之所以研究这个是因为看到下面一段代码，大家可以猜猜它的作用是什么（应该有不少人都见过~）：

```javascript
function arrayGenerator(length) {
  return Array.apply(null, { length: length }).map(Number.call, Number)
}
```

