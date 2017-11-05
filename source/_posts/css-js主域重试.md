---
title: webpack 项目 css/js主域重试
date: 2017-08-27 12:43:42
tags:
    - 前端构建
---

为了提高网站的访问速度，现在一般会将静态资源放在 CDN 下，而不是放在网站的域名之下。以腾讯课堂为例，其域名为 ke.qq.com，打开控制台，访问 ke.qq.com，我们可以看到 js 文件放在了 CDN 7.url.cn 下，css 文件放在了 CDN 8.url.cn 下。尽管 CDN 的服务可用性一般宣称 99.9% 甚至 99.999%，然而实际上监测结果比该数值要小一些。为了应对这种情况，需要做到当发现 css 或 js 文件从 CDN 加载失败时，能再次从网站的域名加载。

<!--more-->

可以将“发现 css 或 js 文件从 CDN 加载失败时，能再次从网站的域名加载“”这个目标分解成四个问题来解决：

1. 如何判断 css 文件加载失败？
2. 如何从主域再次加载 css 文件？
3. 如何判断 js 文件加载失败？
4. 如何从主域再次加载 js 文件？

接下来将会就这四个问题，对使用 webpack 打包的项目进行具体的讨论。

不过在具体讨论之前，先补充一个知识点：webpack 打包生成所有文件后，会触发 'done' 事件。我们可以通过监听 'done' 事件，然后对 css 和 js 文件做包装，对 html 做 js 代码注入等。


## css 主域重试

css 的作用就是改变元素的样式，从这一点出发，我们可以想出如下的方案：

1. 首先向 css 文件添加一条规则；
2. 接着向 html 文件中添加一个元素，最后通过 js 判断第一步中添加的规则是否起作用：
   - 若起作用，则说明 css 加载成功；
   - 若未起作用，则说明 css 加载失败，需要从主域重试。

这种方案可以形象地叫做“**埋点**”：向 html 文档中埋入一个检查点。

通过一个具体的例子来看看如何实现。首先，假设有如下 css 代码：

```Css
/* old css_example1.css */
.elvin {
  font-size: 16px;
}
```

然后，向其中添加一条规则：

```css
/* new css_example1.css */
#css_example1 {
  display: none;
}

.elvin {
  font-size: 16px;
}
```

最后，向 html 文件中添加一个 id 为 `css_example1` 的 div，并通过 js 来做判断：

```html
<div id="example1"></div>
<script>
  var div = document.getElementById('example1');
  if (getComputedStyle(div).display !== 'none') {
    // 说明 css 加载失败
    var newLink = document.createElement('link');
    newLink.rel = 'stylesheet';
    newLink.href = newUrl; // 主域下该 css 对应的地址
    document.head.appendChild(newLink);
  }
</script>
```

插入的 js 逻辑为：当判断出 css 文件加载失败后，创建一个新的 link 标签，然后将其地址指向相应的主域地址，最后将其添加到 html 的头部即可。 

> 需要说明的是，上述向 css 添加规则和向 html 注入代码都是通过监听 webpack 的 'done' 事件进行的自动操作，并不需要手动的去插入这些代码。

## js 主域重试

js 主域重试比 css 主域重试要复杂很多，因为 js 之间往往会存在依赖关系，所以对 js的执行顺序有着要求。举例来说，假设在 html 中依次引入了 1.js， 2.js， 3.js，那么我们希望最终能实现如下效果：

- 若文件1.js，2.js，3.js 正常加载，则每一个 js 加载成功后立即执行；
- 若文件1.js，3.js 正常加载， 2.js 加载失败，则 1.js 加载成功后立即执行，待 2.js 重试成功，再按序运行2.js，3.js；
- 若文件1.js，2.js，3.js 均加载失败，则全部重试，若都成功，再按序运行 1.js，2.js，3.js。

也就是说，认为 2.js 是 依赖于 1.js，3.js 是依赖于 1.js 和 2.js，所以必须保证按照 1.js，2.js 和 3.js 的顺序来执行。这一想法是符合用 webpack 打包的项目的实际情况的：使用 webpack 打包的项目每个页面一般引入三个 js 文件：

1. vendor.js：整个项目的基础库打包成该文件；
2. common.js：将多个（大于等于3）页面公用的库打包成该文件；
3. xxx.js：页面涉及的不包含在前面两个文件中的代码。

上述三个文件必须按照 vendor.js，common.js 和 xxx.js 的顺序来执行。

### 理想情况

在不需要考虑兼容性的情况下，js 主域重试其实也很好实现：监听 script 标签的 onerror 事件，假若发现 js 加载失败，则通过 `document.write()` 方法，立即写入一个新的 script 标签，该标签的 src 指向主域地址。这种方法的神奇之处在于 `document.write()`，通过它写入的 script 新标签，会阻塞后续 script 脚本的执行，直到新标签加载并执行完毕。

这种方法简直完美，实现代码也不超过 10行，然而现实是它不仅仅在 IE 上不能正常工作，在 Edge 上也不行：对于 windows 家的浏览器，哪怕 document.readystate 是 loading，在事件响应函数中调用 `document.write()` 也会造成整个 html 的清空覆盖。所以，必须另寻它法。

接下来将具体讲一讲我所想到的 webpack 项目中 js 主域重试的解决方案，和大家一起讨论。

### js 加载成功的判断

网上现有的资料大部分都是通过 script 标签的 `onload`/`onerror` 事件来判断 js 的加载成功与否，有时为了兼容低版本的 IE，还需要通过 script 标签的 `onreadystatechange` 事件来判断。感谢 webpack 提供了在不修改源文件的情况下对打包出来的 js 做注入的功能，所以类似于 css 埋 div 的方法，也可以在 webpack 构建的时候，向 js 文件埋入变量，然后尝试访问该变量，若得到值，则说明 js 文件加载成功；若未得到值，则说明 js 文件加载失败。

假设有一个 js 文件 js_exampl1.js 如下：

```javascript
// old js_example1
console.log('js_example1');
...
```

那么可以根据文件名，埋入一个唯一的变量：

```javascript
// new(1) js_example1
IMWEB_WEBPACK.js_example1 = true;
console.log('in js_exampl1');
...
```

最后，再根据这个变量来进行加载成功与否的判断：

```javascript
if (!IMWEB_WEBPACK['js_example1']) {
  // 说明 js 加载失败
  var newScript = document.createElement('script');
  newScript.src = newSrc; // 主域下该 js 对应的地址
  document.body.appendChild(newScript);
}
```

> 上述代码有两点需要说明一下：
>
> 1. 用文件名作为变量埋入是因为 webpack 打包后的文件名都含 md5 值，可以保证唯一性；
> 2. 埋入的变量都放在 IMWEB_WEBPACK 下是为了避免污染全局变量。

### js 避免立即执行

本节一开始有谈到，假如引入了1.js， 2.js， 3.js，若文件1.js，3.js 正常加载， 2.js 加载失败，则希望 3.js 在 2.js 从主域加载成功后再执行。为了实现这个需求，需要 3.js 在加载成功后，原代码不立即执行，我的实现方式是将原来的代码用函数体包裹起来避免立即执行，然后再调用一个事先写好的函数进行判断。

还是举例来进行具体说明。对于上一小节的 js_example1.js 文件，继续做如下改造：

```javascript
// new(2) js_example1
IMWEB_WEBPACK.js_example1 = true;
function IMWEB_WEBPACK_js_exampl1() {
  console.log('in js_exampl1');
  …
}
IMWEB_WEBPACK_JS_ONLOAD('js_exampl1');
```

原 js_example1 的代码被包裹在函数 `IMWEB_WEBPACK_js_exampl1` 中从而避免了立即执行。该函数名可在构建时自动生成，具体规则为 `IMWEB_WEBPACK_` + 文件名。然后将文件名传入 `IMWEB_WEBPACK_JS_ONLOAD`，做下一步操作。

### js 执行顺序保证

为了实现 js 主域重试，还需要向 webpack 生成的 html 文件插入两段 js 代码，第一段代码需要插入在所有外联的 js 代码之前，具体如下：

```javascript
IMWEB_WEBPACK.JSARRAY = [
  { name: 'js_example1', url: '//7.url.cn/js_example1.js'},
  // ...
];
IMWEB_WEBPACK.firstLoad = true; // 标志是否是从页面本身的 script 标签加载
IMWEB_WEBPACK.jsRunCnt = 0; // 计数器：统计已运行的 JS

function IMWEB_WEBPACK_JS_ONLOAD (name) {
  if (IMWEB_WEBPACK.firstLoad) {
    // 从本有的 script 标签加载的 JS 
    var jsFile = IMWEB_WEBPACK.JSARRAY[IMWEB_WEBPACK.jsRunCnt];
    if (jsFile.name === name) {
      jsFile.isLoad = true;
      window['IMWEB_WEBPACK_' + jsFile.name]();
      IMWEB_WEBPACK.jsRunCnt++;
    }
  }
  else {
    // 从新添加的 script 标签加载的 JS
    IMWEB_WEBPACK.jsLoadedCnt++;
    if (IMWEB_WEBPACK.jsLoadedCnt === IMWEB_WEBPACK.JSARRAY.length) {
      IMWEB_WEBPACK_RunScripts();
    }
  }
}
```

上述代码逻辑较为简单，做一点说明：

- `IMWEB_WEBPACK.JSARRAY` 是扫描 html 文件得到的，它记录了该 html 所引入的所有外联 js 的文件名和链接；
- `IMWEB_WEBPACK.firstLoad` 用于记录整个页面的 js 加载状态：当所有外联 script 标签还未尝试加载完时，值为 true；当已尝试加载完时（无论成功与否），值为 false；
- `IMWEB_WEBPACK.jsRunCnt` 用于统计已经加载并成功运行的 js 文件个数；
- `IMWEB_WEBPACK_JS_ONLOAD` 每一个外联的 js 加载成功后都会调用这个函数，当所有外联 script 标签还未尝试加载完时，若尚未有 js 加载失败，则每一个 js 加载成功后函数体都会立即执行；否则不执行。



第二段代码需要插入在所有外联的 js 代码之后，具体如下：

```javascript

```

上述代码会在所有外联 script 标签尝试加载后（无论成功与否）执行，它主要负责重试从 CDN 加载失败的 js，并在所有主域重试的  js 加载成功后执行尚未执行的 js 脚本。



## 效果演示

![css 主域重试演示](http://ww1.sinaimg.cn/large/006b78jTgy1fiye371j4oj30ms04kwf7.jpg)

在上图中，可以看见 common_md5.css 从 8.url.cn 加载失败后，自动从主域再次尝试拉取。

![js 主域重试演示](http://ww1.sinaimg.cn/large/006b78jTgy1fiye4djme0j30mn06q3zm.jpg)

在上图中，可以看见 vendor_md5.js 从 7.url.cn 加载失败后，自动从主域再次尝试拉取。需要注意的 vendor_md5.js 从 7.url.cn 尝试拉取了两次，这应该是 Chrome （版本 60）本身的失败重试机制。

## 总结

css 主域重试较为简单，核心概念就是**埋点**；js 主域重试则较为复杂，因为涉及到了依赖的解决问题，核心在于**埋变量**和通过 `jsRunCnt`、`jsLoadedCnt` 两个计数器进行相应的判断。

有说的不清楚的地方或者读者认为有待商榷的地方欢迎在评论区指出，大家一起来进行讨论。