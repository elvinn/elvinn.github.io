---
title: ts-check 立即上手，JSDoc 添加类型
date: 2017-10-03 23:27:21
tags:
    - TypeScript
---

由于 JavsScript 是弱类型，所以在大型项目中使用时显得能力略有不足。从七月份在腾讯实习到现在，接触到了不少项目的代码，平均算来每天都有 70% 的时间用于阅读、理解他人的代码。每次阅读他人代码的时候，我心中都会冒出来两个强烈的愿望：要是 JavaScript是强类型的多好！要是文档能再详细一点就好了！多亏了 TypeScript 和 JSDoc，这两个愿望都有变成现实的可能。

<!--more-->



## @ts-check 立即上手

使用 TypeScript 的最佳方式肯定是直接使用它的语法来编写 .ts 文件，然后通过编译器转换成 .js 文件。然而对于老项目而言，切换构建往往意味着巨大的风险，而且假如将来 JavaScript 也引入了类型系统（这非常有可能），那又得从 TypeScript 切回 JavaScript （回归标准）。那么有没有一种无痛的方式，让我们既可以享受 TypeScript 带来的好处，又能不改变项目的现有构件方式呢？

答案就是 `// @ts-check`，在 .js 文件的头部引入这样一行注释，就可以使用 TypeScript 了。

举个例子，在下图中我们首先声明了一个变量 a，然后把数字 1 赋给了它，接着又把字符串 '123' 覆盖了它，看起来好像没有什么问题。

![未使用 @ts-check](https://ws1.sinaimg.cn/large/006tNc79ly1fk5i18mhvdj30f8066glt.jpg)

现在让我们加上`// @ts-check`，咦，怎么 a 下面出现了红色的报错？把鼠标移到 a 处，发现报错是"*Type '"123'" is not assignable to type 'number'*"，也就是说在 TypeScript 中这种把字符串 '123' 赋值给数字变量 a 的做法是不妥的。

![使用 @ts-check](https://ws2.sinaimg.cn/large/006tNc79ly1fk5i415y3tj30f207u74k.jpg)

享受 TypeScript 的好处就是这么简单，不需要改变构建，不需要进行项目的迁移，所需要做的仅仅是在 .js 文件的头部加入 `// @ts-check`（前提是你使用的是 VS Code，不过其它的编辑器下载相应的插件即可）。

## JSDoc 添加类型

如果仅仅使用 `// @ts-check`的话，我们只能使用它的自动类型推断功能，这对于大型项目来说是远远不够的，我们希望能像强类型语言一样指定每个变量的类型。本着不对项目产生侵入的原则，TypeScript 可以通过 JSDoc 风格的注释来完成这一点。接下来的举例说明取自[官方的文档](https://github.com/Microsoft/TypeScript/wiki/JSDoc-support-in-JavaScript)：

```javascript
/**
 * 使用 "@type" 来声明类型
 * @type {string}
 */
let var1;

/** @type {Window} */
let var2;

/**
 * 用 “return” 说明函数的返回值类型
 * @return {number}
 */
function fn1() {}

/**
 * 可以像使用 "@return" 一样使用 "@returns"
 * @returns {{a: string, b: number}}
 */
function fn2() {}

/**
 * 可以指定 union 类型，如字符串或者布尔值
 * @type {(string | boolean)}
 */
let var3;

/**
 * 声明元素类型是数字的数组 - 方式1
 * @type {number[]}
 */
let var4;

/**
 * 声明元素类型是数字的数组 - 方式2
 * @type {Array.<number>}
 */
let var5;

/**
 * 声明元素类型是数字的数组 - 方式3
 * @type {Array<number>}
 */
let var6;

/**
 * 声明对象类型
 * @type {{a: string, b: number}}
 */
let var7;

/**
 * 用 "@typedef" 自定义复杂类型
 * @typedef {Object} SpecialType - 创建一个新的类型 'SpecialType'
 * @property {string} prop1 - SpecialType 属性 prop1 是 string 类型
 * @property {number} prop2 - SpecialType 属性 prop2 是 number 类型
 * @property {number=} prop3 - SpecialType 属性 prop3 是可选的 number 类型
 * @prop {number} [prop4] - SpecialType 属性 prop4 是可选的 number 类型
 * @prop {number} [prop5=42] - SpecialType 属性 prop5 是可选的 number 类型（默认值 42））
 */
/** @type {SpecialType} */
let specialTypeObject;


/**
 * 声明函数参数类型
 * @param p0 {string} - TS 风格声明 p0
 * @param {string}  p1 - p1 是 string 类型参数
 * @param {string=} p2 - p2 是可选的 string 类型参数
 * @param {string} [p3] - 另外一种可选参数写法
 * @param {string} [p4="test"] - p4 是可选的 string 类型参数（默认值为 "test"）
 * @return {string} - 函数返回值是 string 类型
 */
function fn3(p0, p1, p2, p3, p4){
  // TODO
}

/**
 * 也可以使用模板来声明类型
 * 如 fn4 表示返回值和参数 p1 是相同类型
 * @template T
 * @param {T} p1
 * @return {T}
 */
function fn4(p1){}
```

## 写在最后

对于老项目，使用 `// @ts-check` 和 JSDoc 引入 TypeScript 来享受类型系统的好处是最简单、学习成本最低的方法。对于新项目，相较于激进地使用 .ts 文件，我认为 `// @ts-check` 和 JSDoc 是更好的方法，因为 JavaScript 在不久的未来很有可能会引入可选的类型系统（类似于Python 3），到时候可以避免再从 TypeScript 回归 JavaScript。



## 参考链接

1. [Type Checking JavaScript Files](https://github.com/Microsoft/TypeScript/wiki/Type-Checking-JavaScript-Files)
2. [JSDoc support in JavaScript](https://github.com/Microsoft/TypeScript/wiki/JSDoc-support-in-JavaScript)

