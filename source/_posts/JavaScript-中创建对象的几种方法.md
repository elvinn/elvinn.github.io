---
title: JavaScript 中创建对象的几种方法
tags:
  - 读书笔记
  - 前端
  - JavaScript
date: 2017-01-19 13:35:59
---

在 JavaScript 中，面向对象编程的概念和 C++，Java 等语言都不太一样。在其他的语言中，面向对象的编程需要区分两个概念：**类**和**实例**，**类**是**实例**的模板，**实例**是根据**类**创建出来的对象。以 C++ 为例，在`Student jack = new Student();`  中，`Student` 为类，`jack` 为实例。不过在 JavaScript 中，不区分类和实例，而是通过 `原型（prototype）` 实现面向对象的编程。关于原型的介绍不是这篇文章的重点，想要深入了解的朋友可以参考 [MDN - 继承与原型链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)。接下来将主要讲讲 JavaScript 中创建对象的几种方法和其优缺点。

<!--more-->

## 工厂模式

工厂模式抽象了创建具体对象的过程，将创建的具体逻辑和过程交由工厂函数进行处理，一个简单的例子如下所示：

```javascript
var createPerson = function (name) {
  var o = {};
  o.name = name;
  o.sayName = function () {
    console.log(o.name);
  };
  return o;
};
var jack = createPerson(‘jack’);
```

工厂模式的一个主要问题是我们没有办法判断 `jack` 的对象类型：

```javascript
jack.constructor === createPerson; // false
jack instanceof createPerson; // false
```

不知道 `jack` 是一个人，还是一个狗狗的话就是一个大问题了，我们没办法针对性地采取行为，要是一不小心把他当成了小狗，喂给他一根骨头就要友尽了。

## 构造函数模式

既然有了问题我们就要解决它，所以有了构造函数模式：通过自定义构造函数，再使用 `new` 操作符进行对象创建即可以将 `jack` 标识为具体的类型：

```javascript
// 定义构造函数 Person
var Person = function (name) {
  this.name = name;
  this.sayName = function () {
    console.log(this.name);
  };
};
var jack = new Person(‘jack’);
```

这种方法的关键就是弄清楚 `new` 具体完成了哪些行为，实际上会经历下面 4 个步骤：

1.  创建一个新的对象，并将其 `[[Prototype]]` 指向构造函数的 `prototype` 属性；
2.  将构造函数中的 `this` 指向上一步中创建的对象；
3.  执行构造函数中的代码；
4.  如果构造函数返回值是原始数据类型（没有 return 语句时返回值为 `undefined`)的话，第一步中创建的对象会被返回；如果构造函数返回的是一个对象，则第一步中创建的对象会被抛弃。

通过上述方法，我们就可以判断 `jack` 的对象类型了：

```javascript
jack.constructor === Person; // true
jack instanceof Person; // true
jack.proto === Person.prototype; // true
Object.getPrototypeOf(jack) === Person.prototype; // true
```

不过在实际使用构造函数模式的时候，还存在着两个问题：

*   将构造函数当成了普通函数使用，从而污染了全局作用域：

    ```javascript
    var jack = Person(‘jack’); // 没有使用 new
    window.sayName(); // 输出 “jack”
    ```

为了避免出现这种情况，一种解决方法是将构造函数的首字母大写，提醒使用者这是一个构造函数。

*   每个函数都要在实例上重新创建一遍，既浪费了空间，也减慢了构造函数的执行速度。还是举个例子来说明吧：

    ```javascript
    var jack = Person(‘jack’); // 没有使用 new
    window.sayName(); // 输出 “jack”
    ```

尽管实际上 `bob` 和 `jack` 的的函数 `sayName` 是一样的，但是构造函数模式还是分别为它们分配了空间，这样就造成了不必要的浪费。

## 原型模式

原型模式和构造函数类似，但是通过将对象的函数绑定到其`prototype` 对象上，从而让所有对象实例共享它所包含的属性，进而解决了刚刚提到的构造函数模式中的第二个问题。一个简单例子如下：

```javascript
var Person = function (name) {
  this.name = name;
};
Person.prototype.sayName = function () {
  console.log(this.name);
};
var jack = new Person(‘jack’);
var bob = new Person(‘bob’);
```

根据刚刚在构造函数模式中介绍的关于 `new` 执行动作可以知道，所有通过 `new Person()` 创建出来的对象其原型均指向 `Person.prototype` 属性所指向的对象。

![原型模式示意图](http://ww3.sinaimg.cn/large/006tNbRwjw1fasid90p02j30ls0brq3b.jpg)

当我们调用 `jack.sayName()` 时，执行器会先在 `jack` 这个对象中进行查找，未查找到后便根据原型链到上一级进行查找，在 `某个对象`中查找到后就执行函数，这里的 `某个对象` 就是 `Person.prototype` 所指向的对象，它的属性 `constructor` 指向 `Person` 对象。

我们来测试一下：

```javascript
jack.sayName === bob.sayName; // true
```

开心，和预期的一样。

## 动态原型模式

不知道大家看到原型模式会不会有些不舒服，反正我是有的：把对象的属性定义和函数定义分开来让我感觉像是把一个完整的对象给割裂了，怎么看怎么不舒服。还好像我这样有强迫症的人不在少数，所以便有了动态原型模式— 将对象的函数定义也包装在构造函数中：

```javascript
var Person = function (name) {
  this.name = name;
  if (typeof this.sayName !== “function”) {
    // 仅会在第一次执行构造函数时运行此段
    Person.prototype.sayName = function () {
      console.log(this.name);
    };
  }
};
var jack = new Person(‘jack’);
```

这样看起来就舒服多了嘛，不过因为会在构造函数中调用 if 进行判断，所以会有一点点的性能损失。

## 寄生构造函数模式

这种模式的基本思想是创建一个函数，该函数的作用是仅封装创建对象的代码，然后再返回新创建的对象，但从表面上来看又像是典型的构造函数，下面是一个例子：

```javascript
var Person = function (name) {
  var o = {};
  o.name = name;
  o.sayName = function () {
    console.log(o.name);
  };
  return o;
};
var jack = new Person(‘jack’);
```

将它和工厂函数进行对比，可以发现它除了使用时通过 `new` 操作符外，与工厂模式其实是一模一样的，我觉得之所以要加上 `new` 然后新命名一个名字，只是为了让使用者清楚这是新类型的对象吧。。。。（囧）

之所以叫做**寄生**构造函数模式，是因为这个模式可以在不方便直接修改已有的对象的前提下，创建与已有对象类似，但是具有额外方法的特殊对象，就像是个寄生虫一样：

```javascript
var SpecialArray = function () {
  // 创建已有的对象
  var values = new Array(); 
  values.push.apply(values, arguments);
  // 在已有的对象的基础上进行改造
  values.toPipedString = function(){
    return this.join(“|”);
  }
  return values;
}
var colors = new SpecialArray(‘red’, ‘blue’, ‘green’);
colors.toPipedString(); // ‘red|blue|green’
```

通过上述代码，在不影响其他 `Array` 对象的前提下，我们创建了一个具有特殊方法 `toPipedString` 的特殊数组。

## 稳妥构造模式

上面所有的模式都存在一个问题：对象的所有属性可以被外部任意的修改和访问，这个问题在某些安全的环境中会显得格外严重。大名鼎鼎的 Douglas Crockford 因此发明了**稳妥对象（Durable Objects)**这个概念，即没有公共属性，而且其方法也不引用 this 的对象。创建稳妥对象的模式被称之为稳妥构造函数模式，将前面的例子重写如下：

```javascript
var createPerson = function (p_name) {
  var o = {};
  // 定义外部可访问的变量
  o.type = ‘Person’;
  // 定义私有变量
  var name = p_name;
  o.sayName = function () {
    console.log(name);
  };
  return o;
};
var jack = createPerson(“jack”);
jack.type; // ‘Person’
jack.name; // undefined
```

## 写在最后

在上面提到的各种模式中，原型模式是使用最广泛，认同度最高的一种创建自定义类型的方法，不过其它的模式也有其用物之地，我们应该根据它们的优缺点进行恰当的选择。

同时，在 ES6 中引入了 `class` 这个概念，让 JavaScript 也能像我们在开头所示的其它语言一样，通过类似的语法创建类与对象：

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  
  sayName() {
    console.log(this.name);
  }
}
var jack = new Person(‘jack’);
jack.sayName(); // ‘jack’
```

不过尽管写法上同其它面向对象的语言类似，但是本质却还是 JavaScript 通过原型（prototype）来创建对象。

## 参考资料

1.  [stack overflow - How does the new operator work in JavaScript?](http://stackoverflow.com/questions/6750880/how-does-the-new-operator-work-in-javascript)。
2.  《JavaScript 高级程序设计（第3版）》 6.2 节-创建对象。