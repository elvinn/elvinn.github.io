---
title: 怎样让开源项目看起来“高大上”
date: 2018-01-28 15:43:17
tags:
	- 基础知识
typora-copy-images-to: ipic
---

为了避免重复造轮子 ，我们往往会借助开源的项目实现一些功能。很多时候，选择使用哪一个开源项目就像选择男、女朋友一样，固然内在很重要，但是眼缘也很关键，只有看对了眼，才会进一步地了解。作为开源项目的开发者，当然是希望自己写出来的成果能被更多的人尝试使用，所以这篇文章主要谈一谈怎样让开源项目看起来“高大上”，从而让别人有使用的冲动。

<!--more-->

首先要弄清楚一个问题：怎样的开源项目才算“高大上”呢？这里举出两个例子，一个是在 2017 年越来越火的前端框架 [Vue](https://Github.com/vuejs/vue)，还有一个是我所在团队大神 avwo 开源的 web 调试代理工具 [whistle](https://Github.com/avwo/whistle)，大家可以去他们的 Github 地址感受一下，下面展示一下他们 Readme 的开头部分：

![Vue Readme Head](https://ws3.sinaimg.cn/large/006tNc79ly1fnwfy38nsgj30q909j0tm.jpg)

![whistle Readme Head](https://ws1.sinaimg.cn/large/006tNc79ly1fnwfyugbpmj30qj0g50ws.jpg)

在我个人看来，一个“高大上”的 Github 上的开源项目应该满足这些条件：

- 一句话说明项目的功能；
- 有相对完善的测试用例和较高的代码覆盖率；
- 通过徽章明确地指出项目的兼容性、最新版本、被使用情况、License 等；
- 有详尽地 `CHANGELOG` 或者 `release notes`，也就是更新说明；
- 有简介、明了的 commit 信息；
- 提供 discord、Telegram（国外）、QQ 群或者微信群（国内）等供使用者交流的地方；
- 利用 Github 能力提供 issue 和 pull request 的模板。


> 注意：上述排名分先后，如果有能力提供有一个美观、独特的 Logo，当然更好 🙃。



接下来会以我最近刚写的一个小项目 [git-master-merged](https://Github.com/elvinn/git-master-merged/) 为例，说一说如何利用 Github 和相关平台的能力实现上面提到的内容，说的不好或者错误的地方欢迎大家指出来。



## 简洁的说明

![42A631BA-A9D8-4ED6-AD88-46C1036FF08A](https://ws4.sinaimg.cn/large/006tNc79ly1fnwiuuxe7sj30sb03nglx.jpg)

在每一个 Github 项目的最上方，都有一小块地方让我们来填写项目的描述、相关链接和对应的主题。也许是因为地儿太小了，大家没注意，这里都是空白的一片。不过不能因为块头小就小瞧呀，那潘长江还活不活了！悄悄地告诉大家，**Github 的搜索功能就是根据这里的描述和主题 进行的**，所以这里最好能一句话讲清楚项目的功能和作用，从而既能方便别人快速了解，也能让更多人搜索到该项目，例如这样：

![简洁的说明举例](https://ws1.sinaimg.cn/large/006tNc79ly1fnwjnh6vcyj30pp0403yy.jpg)

> 现在越来越多的项目喜欢在描述的开头加上一个 emoji 表情，可以参考[这份 emoji 列表](https://gist.Github.com/rxaviers/7360908)，将形如 `:xxxx:` 的短语粘贴即可展示成表情。 

## 完善的测试

“可靠性”这个词如今被频繁的提起，假如一个开源的项目没有任何测试代码，那么怎么能奢求别人在生产环境中使用呢？各个语言都有不同的测试框架，如 JavaScript 的 mocha、jest，Python 的 unittest 等，基本用法和概念都相似，这里就不赘述了。这儿主要谈谈**持续集成（Continuous integration，简称CI）**在开源项目中的应用。

由于开源项目迭代速度较快，而且经常会收到别人的 pull request，所以如何在快速迭代中，保持较高的质量成为了一个重要的问题。通过持续集成，我们可以在自己 commit 或者在接收他人 pull request 时，自动触发测试代码的执行，从而能通过测试结果快速、直观地发现错误，让质量得到保障。

[git-master-merged](https://Github.com/elvinn/git-master-merged/) 项目所使用的持续集成工具是 [Travis CI](https://travis-ci.org/)，对于 Github 上的开源项目，可以免费使用。使用起来也非常简单，一共只有四步：

1. 用 Github 账号登录 https://travis-ci.org/；
2. 选择要使用 Travis CI 的项目：
  ![8689F10C-5009-4C77-A28A-89BB684C8E24](https://ws1.sinaimg.cn/large/006tNc79ly1fnwkxnrw6ij309401ajr8.jpg)
3. 在项目中添加 `.travis.yml`，一般在该文件中指定语言和测试环境；
4. commit 刚刚作出的修改，并推向 Github 仓库。

（更详细的说明可以参考 [官方文档](https://docs.travis-ci.com/user/getting-started)）

假如一切顺利的话，就可以在 Travis CI 的后台看到通过的结果，从而使用 `build: passing` 的徽章：

![50F590E1-BAD0-4836-BDB6-2EEEA5843CB9](https://ws1.sinaimg.cn/large/006tNc79ly1fnwl5fd4kcj30t3099t9w.jpg)

除了测试用例是否通过外，测试代码的覆盖率也是一个很重要的指标。我们也可以通过持续集成的方式，在 `.travis.yml` 文件中添加相关字段的说明，从而在 [codecov](https://codecov.io/) 等网站上自动检测 diamante 覆盖率，从而再领取一枚徽章。

## 个性化的徽章

![86E75169-98C2-44E6-BD47-2D140FC761BF](https://ws2.sinaimg.cn/large/006tNc79ly1fnwlauzkemj30qh05kt9j.jpg)

看到这些勋章是不是心里就有点痒痒的，想尽快为自己的项目也添加上？这里强烈推荐 http://shields.io/ 这个网址，通过它，我们可以为项目添加上各种各样的徽章，例如：

- 项目的最新版本；
- 项目的被使用情况；
- 项目的兼容情况；
- 测试是否通过以及代码覆盖率情况；
- 代码质量和风格情况。

如果上述这些功能都不能满足你的话，还可以自定义专属于自己的勋章！这里有一篇 [GitHub 项目徽章的添加和设置](https://lpd-ios.github.io/2017/05/03/GitHub-Badge-Introduction/) 详细介绍的文章，我就不多说了，大家赶快用起来吧 :smile:

## 规范的提交记录和更新说明

规范的提交记录和更新说明，既可以让使用者清楚地知道更新的内容从而有更强的意愿进行升级，也可以让项目参与者了解项目的演进历史和当前状况从而更好地进行后续开发。再者，规范的提交记录，可以在出现问题时，快速地定位，找到错误代码，从而解决问题（也可以更快地知道是谁犯了错👻）。

当前社区中使用较多的 commit 提交规范是 `Angular` 规范，英文文档可以阅读 [Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.greljkmo14y0)，中文详尽的介绍可以阅读 [Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)，这里简要地介绍一下：

commit 的格式包含 `Header`、`Body`、`Footer` 三个部分，形如：

```
<type>(<scope>): <subject>

<body>

<footer>
```

其中，`Header` 是必须，`Body` 和 `Footer` 可以省略。进一步，`Header` 中，说明影响范围的 `scope` 可以省略，但是表示种类的 `type` 和 `subject` 必须包含，其中 `type` 只能从下列 7 中标识选取：

- feat：新功能
- fix：修补 bug
- docs：文档
- style： 格式化代码
- refactor：重构
- test：完善测试
- chore：其它维护相关更改

人总是不可靠的，有了规范之后，我们可以通过相关工具来提交和检查提交记录，并自动生成更新说明：

- 使用 [commitizen](http://commitizen.github.io/cz-cli/) 来进行交互式的 commit 提交，从而减少不规范的情况；
- 使用 [commitlint](http://marionebl.github.io/commitlint/#/) 来检查不规范的 commit 提交，从而提出不规范的记录；
- 使用 [conventional-changelog](https://github.com/conventional-changelog/https://github.com/conventional-changelog/conventional-changelog) 来自动地根据 commit 中 `feat`、`fix`、`Performance Improvement` 和 `Breaking Changes` 生成更新说明，而且是增量更新，并不会覆盖已有的文件内容。


这里通过几张图片了解一下：

![commitizen](https://github.com/commitizen/cz-cli/raw/master/meta/screenshots/add-commit.png)

![commitlint](https://cdn.rawgit.com/marionebl/commitlint/3594397919c6188ce31ccfc94a0113d625d55516/docs/assets/commitlint.svg)

![conventional-changelog](https://ws4.sinaimg.cn/large/006tNc79ly1fnwow60htrj316g0iggpd.jpg)

## 清晰的模板

 对于开源项目而言，他人给项目提 issue 或者 pull request 都是十分频繁的，作为项目的开发者或者维护者，最希望他人提供的信息是简洁、明了且有用的。通过 github 提供的能力，我们可以编写一套模板，让他人在使用时直接往其中填入内容即可。整个过程也十分简单：

1. 在项目根目录下创建目录 `.github`；
2. 编写文件 `.github/issue_template.md` 文件：
```
### Expected behavior


### Actual behavior


### Steps to reproduce the behavior
```
3. 变更提交至 github 并观察效果：


![github issue templates](https://ws4.sinaimg.cn/large/006tNc79ly1fnwp4lt8xdj314a0r0ade.jpg)

> 官方文档可以参考：
>
> - [Creating an issue template for your repository](https://help.github.com/articles/creating-an-issue-template-for-your-repository/)
> - [Creating a pull request template for your repository](https://help.github.com/articles/creating-a-pull-request-template-for-your-repository/)

## 写在最后

还是拿谈恋爱举例子吧，“外在决定两个人在一起，内在决定两个人在一起多久”，一个看起来“高大上”的门面当然可以吸引更多的人来使用我们的项目，但是只有项目本身具有较高的价值并积极运营时，才能留下用户，让项目更好地运作下去。

希望开源社区能发展地越来越好！

  