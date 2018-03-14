---
title: 仅需四步，即可体验 Service Worker 带来的快感
date: 2018-03-04 10:47:53
tags:
	- 工具
	- 前端
---

随着 17 年年底苹果宣布 Safari 支持 Service Worker，越来越多的网站势必会在生产环境中引入它。不过由于存在接入的成本，大家无法立马体验到 Service Worker 应用在自己网站上所带来的优点。为了能快速体验到 Service Worker 带来的丝滑快感，趁着春节后上班第一周的空闲时间，我开发了一款 [whistle.service-worker](https://github.com/elvinn/whistle.service-worker) 的小工具，只需四步就可以为想要使用 Service Worker 的网站测试效果。

<!--more-->

## 举个例子

假设我们想测试 [Github Help](help.github.com) 使用 Service Worker 后的效果，那么只需如下四步:

### 1. 安装插件

[whistle.service-worker](https://github.com/elvinn/whistle.service-worker) 是基于 [whistle](https://github.com/avwo/whistle) 制作的一款插件，所以需要同时安装它们两个，执行如下命令进行安装并启动即可：

```shell
$ npm install --global whistle whistle.service-worker
$ w2 start
```

启动成功后可看到如下提示：

![w1](https://ws4.sinaimg.cn/large/006tNc79gy1fp0sjw00s2j30jo04dmy9.jpg)

### 2. 设置代理

将代理设置为 `127.0.0.1:8899`，如果是 Chrome 的话推荐使用 [SwitchyOmega](https://chrome.google.com/webstore/detail/padekgcemlokbadohgkifijomclgjgif)，Firefox 则推荐使用 [Proxy Selector](https://addons.mozilla.org/zh-cn/firefox/addon/proxy-selector/) 进行设置。设置后结果如下：

![w2](https://ws2.sinaimg.cn/large/006tNc79gy1fp0sjzgy42j30y70enmyx.jpg)

### 3. 配置规则

在浏览器中打开 [配置界面](http://local.whistlejs.com/#rules) 并添加如下规则：

```
help.github.com whistle.service-worker://route=/.*/&strategy=cacheFirst
```

![rule](https://ws4.sinaimg.cn/large/006tNc79gy1fp0sjwmcbcj30qf02pt9f.jpg)

### 4. 打开网站

终于来到最后一步了，只需要打开 [Github Help](https://help.github.com/) 即可发现 Service Worker 注册成功。再次刷新页面，就能享受到它带来的丝滑快感了 O(∩_∩)O~~

![registered](https://ws3.sinaimg.cn/large/006tNc79gy1fp0sjyj3lfj30un0lqq72.jpg)

![fetch](https://ws3.sinaimg.cn/large/006tNc79gy1fp0sjxnil1j30un0lqwkj.jpg)

## 写在最后
实际上，Service Worker 主要是在两个方面发挥作用：

 - 对未加载文件的预拉取能力，例如用户在 A 页面的时候就可以拉取 B、C 页面的文件，从而让用户在页面间的跳转更加流畅；
 - 对已加载文件的缓存控制能力，相较于现有的 HTTP(S) 协议中缓存相关的内容，Service Worker 带来的缓存控制能力不仅粒度更小，而且策略更加丰富。

[whistle.service-worker](https://github.com/elvinn/whistle.service-worker) 工具目前只是在第二个方面做了一点工作，若想完整享受 Service Worker 带来的好处，建议接入 Google 开发的 [Workbox](https://github.com/GoogleChrome/workbox) 工具，功能十分强大👍

> [whistle.service-worker](https://github.com/elvinn/whistle.service-worker) 实际上是基于跨平台的代理工具 [whistle](https://github.com/avwo/whistle) 制作的一款插件，是我**迄今为止用过体验最好、功能最强大的开源、跨平台web调试代理工具**，强烈推荐大家使用。

