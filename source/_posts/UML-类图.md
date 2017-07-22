---
title: UML 类图
tags:
  - JAVA
  - 基础知识
date: 2016-11-17 22:43:34
---

在研究、学习 Android 及第三方库的源码过程中，类本身的属性、方法与类间的关系等都是研究的重点，因而利用统一建模语言（ Unified Modeling Language, UML）提供的类图来展示类的信息十分的重要。

<!--more-->

## [](#类本身的表示 "类本身的表示")类本身的表示

类是对象的蓝图，封装了数据和行为，它是具有相同属性、方法、关系的对象的集合的总称，下面是一个简单的 Person 类的 Java 代码：

<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div><div class="line">11</div><div class="line">12</div><div class="line">13</div><div class="line">14</div><div class="line">15</div><div class="line">16</div><div class="line">17</div><div class="line">18</div><div class="line">19</div><div class="line">20</div></pre></td><td class="code"><pre><div class="line"><span class="class"><span class="keyword">class</span>  <span class="title">Person</span> </span>&#123;</div><div class="line">  <span class="keyword">private</span> String name;</div><div class="line">  <span class="keyword">private</span> <span class="keyword">int</span> age;</div><div class="line"></div><div class="line">  <span class="function"><span class="keyword">public</span> <span class="keyword">void</span> <span class="title">setName</span><span class="params">(String name)</span> </span>&#123;</div><div class="line">    <span class="keyword">this</span>.name = name;</div><div class="line">  &#125;</div><div class="line"></div><div class="line">  <span class="function"><span class="keyword">public</span> String <span class="title">getName</span><span class="params">()</span> </span>&#123;</div><div class="line">    <span class="keyword">return</span> <span class="keyword">this</span>.name;</div><div class="line">  &#125;</div><div class="line"></div><div class="line">  <span class="function"><span class="keyword">public</span> <span class="keyword">void</span> <span class="title">setAge</span><span class="params">(<span class="keyword">int</span> age)</span> </span>&#123;</div><div class="line">    <span class="keyword">this</span>.age = age;</div><div class="line">  &#125;</div><div class="line"></div><div class="line">  <span class="function"><span class="keyword">public</span> <span class="keyword">int</span> <span class="title">getAge</span><span class="params">()</span> </span>&#123;</div><div class="line">    <span class="keyword">return</span> <span class="keyword">this</span>.name;</div><div class="line">  &#125;</div><div class="line">&#125;</div></pre></td></tr></table></figure>

其对应的类图为：
![Person 类图](http://7xlboz.com1.z0.glb.clouddn.com/Person.png)

在上面这个简单的例子中，我们可以看到，从上至下类图分为三个部分，依次为：

*   类的名称
*   类的属性
*   类的方法

其中，属性部分的格式为 `可见性 名称:类型 [=缺省值]`，方法部分的格式为 `可见性 名称(参数类型):返回类型`，可见性的可选项为：

<figure class="highlight plain"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div></pre></td><td class="code"><pre><div class="line">+    public</div><div class="line">-    private</div><div class="line">#    protected</div><div class="line">~    default</div><div class="line">_    static</div></pre></td></tr></table></figure>

## [](#类间关系的表示 "类间关系的表示")类间关系的表示

### [](#泛化（Generalization） "泛化（Generalization）")泛化（Generalization）

泛化，可理解为继承，在 Java 中通过关键字 `extends` 标示，用**带空心三角箭头的实线**表示，例如有如下 Java 代码：

<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div><div class="line"></div><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">B</span> <span class="keyword">extends</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div></pre></td></tr></table></figure>

那么相应的 UML 类图为：
![泛化关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/generalization.png)

### [](#实现（Realization） "实现（Realization）")实现（Realization）

实现，指的是一个类实现接口（可以是多个）的功能，在 Java 中通过关键字 `implements` 标示，用**带空心三角箭头的虚线**表示，例如有如下 Java 代码：

<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line"><span class="class"><span class="keyword">interface</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div><div class="line"></div><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">B</span> <span class="keyword">implements</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div></pre></td></tr></table></figure>

那么相应的 UML 类图为：
![实现关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/realization.png)

### [](#依赖（Dependency） "依赖（Dependency）")依赖（Dependency）

依赖，可简单的理解为类 A 使用到了另一个类 B，可以用 _‘… use a …’_ 进行判断，这种使用关系是具有偶然性的、临时性的、非常弱的。表现在代码层面，就是类 B 作为参数被类 A 在某个方法中使用，用**带燕尾箭头的虚线**表示，例如有如下 Java 代码：

<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div><div class="line">10</div></pre></td><td class="code"><pre><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div><div class="line"></div><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">B</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">  <span class="function"><span class="keyword">public</span> <span class="keyword">void</span> <span class="title">method</span><span class="params">(A param)</span></span>&#123;</div><div class="line">  	<span class="comment">//...</span></div><div class="line">  &#125;</div><div class="line">&#125;</div></pre></td></tr></table></figure>

那么相应的 UML 类图为：
![依赖关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/dependency.png)

### [](#关联（Association） "关联（Association）")关联（Association）

关联，体现的是两个类、类与接口或者接口与接口之间的强依赖关系，可以用 _‘… has a …’_ 进行判断，一般来说是一种长期的稳定的关系。表现在代码层面，就是 B 以属性的形式出现在 A 中，关联既可以是单向的，也可以是双向的，用**带燕尾箭头的实线**表示单向关联，**不带箭头的实现**表示双向关联，例如有如下 Java 代码：

<figure class="highlight java"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div><div class="line">8</div><div class="line">9</div></pre></td><td class="code"><pre><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">A</span> </span>&#123;</div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div><div class="line"></div><div class="line"><span class="class"><span class="keyword">class</span> <span class="title">B</span> </span>&#123;</div><div class="line">  <span class="keyword">private</span> A attribute;</div><div class="line">  </div><div class="line">  <span class="comment">//...</span></div><div class="line">&#125;</div></pre></td></tr></table></figure>

那么相应的 UML 类图为：
![关联关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/association.png)

### [](#聚合（Aggregation） "聚合（Aggregation）")聚合（Aggregation）

聚合，指整体与部分之间的一类特殊的关联关系，是“弱”的包含关系，可以用 _‘… owns a …_ 进行判断，成分类可以不依靠集合类而单独存在，可以拥有各自的生命周期，用**带空心棱形箭尾的实线**表示，例如教室与教室中的学生间的关系，相应的 UML 类图为：
![聚合关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/aggregation.png)

有些时候，我们还需要两个对象在数量上的对应关系（Multiplicity），可以**直接在关联直线上用一个数字或者一个数字范围**来表示，例如有一个最多只能容纳 40 个学生的教室，那么教室和这个教室之间的关系可表示为：
![多重性关联关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/Multiplicity.png)

数字范围表示方法具体为：

*   0..1 没有或者只有一个实例
*   1    有且仅有一个实例
*   0..* 大于等于 0 个实例
*   *    大于等于 0 个实例
*   1..* 大于等于 1 个实例
*   m..n 大于等于 m ，小于等于 n 个实例

### [](#组成（Composition） "组成（Composition）")组成（Composition）

组成，指整体与部分之间一类“强”的包含关系，可以用 _‘… is a part of …’_ 进行判断，成分类必须依靠集合类而存在，两者是密不可分的，整体类负责创建与销毁成分类，当整体类的生命周期结束时也意味着部分类生命周期的结束，用**带实心棱形箭尾的实线**表示，例如人与其大脑间的关系，相应的 UML 类图为：
![组成关系示意图](http://7xlboz.com1.z0.glb.clouddn.com/composition.png)