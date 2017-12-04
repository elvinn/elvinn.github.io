---
title: transform 的副作用
date: 2017-12-03 19:15:53
tags:
    - CSS
    - 前端

---

`transform` 想必大家都很熟悉，可以通过其转换（translate）、旋转（rotate）、缩放（scale）、倾斜（skew）等属性来对元素进行变换，不过这篇文章想探讨的不是这些内容，而是 `transform` 对元素布局、页面渲染方面的影响。例如，你知道它会影响 `fixed` 元素的位置吗？你有想过它会改变元素的层叠顺序吗？

<!--more-->

## tranform 改变 fixed 子元素的定位对象

### 例子探究

首先我们来看一个例子：下面示例中的 fixed 元素设置的是 `top:  -50px`，按理说我们应该是看不见它的，因为它会相对根元素定位到页面上方的外部。然而事实狠狠打了我们的脸，它是可见的！这是为什么呢？

<script async src="//jsfiddle.net/elvinn/p38t336r/1/embed/html,css,result/"></script>

关键就在于这个 `fixed` 元素被拥有 `transform` 的属性的父 div 包裹着，所以**它会相对于这个 `transform` 的父元素定位，而不是我们以为的根元素定位**，又由于父元素有 `margin-top: 50px` 的值，所以两者相抵消（*-50px + 50px = 0*)，最终导致该元素位于页面起始处。

至于为什么会这样，就需要从 W3C 规范中去寻找原因了。在 [W3C - transform rendering](https://www.w3.org/TR/css-transforms-1/#transform-rendering) 中，我找到了这样一段解释：*For elements whose layout is governed by the CSS box model, any value other than `none` for the transform also causes the element to become a containing block, and the object acts as a containing block for fixed positioned descendants*，也就是说 transform 值不为 `none` 的元素会创建一个 **containing block**（作者注：容器块，盒元素定位和大小一般参考容器块进行计算），然后该元素的 `fixed` 子元素会相对该元素进行定位。

### 一点思考

原因搞明白了，那么为什么 W3C 委员会会这样设计呢？依我愚见，可以从两个方面来思考：

1. 假如我们想让 `fixed 元素` 相对根元素进行绝对定位，我们往往会把它作为根元素的第一级子元素，从而也就不会存在它被 `transform 父元素`  包裹的情况了。
2. 那么什么情况下我们会把 `fixed 元素` 放在 `transform 父元素` 下呢？在我看来，只有我们希望它跟随父元素一起变形时才会这样做，要不然为什么不把它放在根元素下呢？

## transform 改变元素层叠顺序

### 例子探究

同样的，我们先来看一个例子：下面示例中第一行为啥都没加的情况下，让第二个元素（蓝色块）通过 `margin-left: -40px` 向左偏移了 40px，按照**后来居上**的层叠规则，它会盖住第一个元素（黄色块）的一部分。第二行给第一个元素（黄色块）加上了 `transform: scale(1)` 后一切就变了，它盖住了第二个元素（蓝色块），**后来居上**的规则貌似不起作用了，这是为什么呢？

<script async src="//jsfiddle.net/elvinn/p9fecmmn/1/embed/html,css,result/"></script>

同样的，还是尝试从 W3C 规范中去寻找原因。在 [W3C - transform rendering](https://www.w3.org/TR/css-transforms-1/#transform-rendering) 中，我找到了一句和上一节基本一样的一句话：*For elements whose layout is governed by the CSS box model, any value other than none for the transform results in the creation of a stacking context*，也就是说 transform 值不为 `none` 的元素会创建一个 `stacking context`（层叠上下文）。层叠上下文，这是什么鬼？好像只听过块级格式化上下文（BFC）。

简单说来，层叠上下文与元素在 z 轴上的展示顺序相关，而且层叠上下文元素的层叠水平要比普通元素高，结合上面的例子来说就是：

1. 根元素是层叠上下文元素，蓝色块和黄色块都是它的子元素；
2. 蓝色块由于 transform 值不为 `none` ，所以升级为层叠上下文元素，因而它的层叠水平比黄色块（普通元素）高。

层叠上下文的内容值得深入地具体探究，这里推荐两个不错内容，一个是 [MDN - 层叠上下文](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context)，另外一个则是 [张鑫旭 - 深入理解CSS中的层叠上下文和层叠顺序](http://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)。



## 写在最后

当使用 CSS 遇到奇奇怪怪问题的时候，我们既可以在 Google 或者 StackOverflow 上寻找答案，也不要忘了 W3C 的存在。有时真相其实早已静静地躺在标准当中，只要我们肯于寻找、善于寻找，定能找到 ^_^

## 参考链接

1. [W3C- CSS Transforms Module Level 1](https://www.w3.org/TR/css-transforms-1/#intro)
2. [张鑫旭 - CSS3 transform对普通元素的N多渲染影响](http://www.zhangxinxu.com/wordpress/2015/05/css3-transform-affect/)
3. [张鑫旭 - 深入理解CSS中的层叠上下文和层叠顺序](http://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)
4. [结一老师 - 视觉格式化模型 - 容器块](http://layout.imweb.io/article/visual-formatting-model.html)