---
title: 苹果 M1 评测
date: 2021-02-01
tags: ["评测"]
categories: ["其他"]
---

先贴配置，内存 16G，硬盘容量 512，系统 macOS Big Sur（11.1）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/0.png)

以下统计截止时间为 2021 年 1 月 31 日。

## 一. 软件兼容性

总结：试了下自己常用的软件，作为日常开发&娱乐电脑跑了两周，没有遇到大问题。

### 1. 社交软件类

目前测试了 钉钉，微信，企业微信，QQ，飞书，如流。看着基础聊天都没问题，具体没有深入体验，初步目测问题不大，而且主流聊天软件肯定会积极适配。

### 2. 开发类

1. JetBrain 家，目前看官网一月份已全线产品支持，据网上的评测资料性能会比之前的兼容版本好很多。以 Webstorm 为例，使用了一周，对一个相对复杂的项目进行了大量重构（抽取函数，文件移动等）操作，并没有出现卡顿，风扇安安静静（比之前的一言不合就风扇呼呼转的小 Intel 要爽太多）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/1.png)

2. VSCode

目前没有看到专门针对 M1 的适配版本，不过从下载最新版本，日常开发了一周，没有遇到卡顿等异常问题，稳定性还不错。

3. homebrew：[https://docs.brew.sh/Installation#macos-requirements](https://docs.brew.sh/Installation#macos-requirements)

官网文档，已支持

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/2.png)

注意事项：如果你以前安装过 intel 兼容版本的 homebrew，可以考虑卸载一下重新装一装，安装路径有一定变更，不再需要 `sudo` 权限去做一些事情了。原文大致翻译：

- `brew install` 不再需要 `sudo` 权限。默认的安装脚本对于  `macOS Intel` 芯片地址为 `/usr/local`，对于 `Apple Silicon` 安装地址改为  `/opt/homebrew`。（如果之前已经安装过能用，但是可能每次需要确认）

关于为什么要修改路径的原因找了找：

Q：Why did Homebrew move from /usr/localto /opt/homebrew?

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/3.png)

p.s. 自从 OSX 10.11 之后，对于系统路径（例如 `/usr/local` ），由于安全策略是不能偷懒去直接改权限的（除非去进行 `csrutil disable` 的操作）。不过作为强迫症并不喜欢去做任何 sudo 的操作，所以当然选择重装

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/4.png)

5. Docker

直接下载正式稳定版是还没有的

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/5.png)

不过找了找文档有一个预览版可以试（不建议在生产环境使用

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/6.png)

6. 其他

- terminal & iTerms：日常使用都没有发现大问题
- SourceTree 通过兼容模式支持，Git 原生支持
- 浏览器：chrome，firefox，360 均能正常使用，苹果自家的 Safari 更加不用说。不过 Chrome 有针对 M1 的版本。

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/7.png)

题外：附带一个 2020 年国内浏览器使用份额表，可以看到二三月份 Chrome 有个很大的下跌，具体新闻有兴趣的小伙伴可以查一查。（所以做兼容的时候是不是可以比起 IE 更需要关注 360 了哈哈）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/8.png)

- 小程序开发IDE：微信小程序，和百度智能小程序均能正常打开，不过手头上没有对应的项目没有继续尝试，有经验的小伙伴可以留言评论。
- MySql，tomcat 等：没有亲自实验，找了些资料据说没问题。

### 3. 小工具类

一些自己日常使用的工具类软件，部分原生支持，部分应该是通过 Rosetta2 兼容。

[Untitled](https://www.notion.so/5e13a6f31f4e4d9f9698ec67c6dace58)

[Microsoft Office 365](https://www.notion.so/Microsoft-Office-365-d03978149a3a454da678f35b960d99fa)

## 二. 性能

不剪视频，因此没有数据对比，就编译速度和 ps 等软件打开速度效率简单做下对比。另外一台电脑为 8G 的 2020 款 macbooks pro，带 touchbar，均为统一网络条件下对比测试。

### 1. 前端 webapck 编译

一个历史项目，十几个入口的多页应用。每次之前的 2016 年款 16G  macbook pro 基本上每次跑起来，然后再开个 webstorm 风扇铁定会转起来，每次本地改个东西热更新都得反应好几秒。有时候开久了做做重构之类的挪挪文件就开始转圈圈，以至于都不大敢开 webstorm，只能用 vscode 靠人脑去建立各种索引。

然后试了下 M1，同时起两个端口，开着 webstorm 和 vscode，各种重构，喊出抽取文件夹挪动基本无压力，风扇安安静静。可以说体验真的有非一般的感觉，提升效率提升生产力提升幸福感的快速直达办法。

不过跑了下编译速度，目前看着并没有太大的差距。公司 intel 芯片的 8G 小内存 MacBook Pro build 花了 15.82s，M1 build 13.61s（基于 `SpeedMeasurePlugin` 插件统计）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/9.png)

### 2. PhotoShop & Sketch

**Sketch**

之前装的老版本发现并不能使用，于是从官网找了找，70 之后的版本据说都可以使用。题外：他们还为了配合苹果 `big sur` 系统的 UI 重新制作了一套全新的 UI（速度真的是可以）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/10.png)

从官网下载了最近版，打开了内部组件库和项目一些内容的稿子，完全没有压力。

但是一般来说我们可能不局限于它本身，比如大多时候会装点小插件，那么他们的兼容性如何。看看作为给程序员最常用的导出 `html` 功能（在新版本的系统以及新版本的加成下  `sketch measure` 插件已经凉凉。不过可以有 Sketch MeaXure 替代品，在 Github 可以直接搜到对应的仓库 & 下载地址：[https://github.com/qjebbs/sketch-meaxure](https://github.com/qjebbs/sketch-meaxure)）

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/11.png)

然而尝试了下，基本上是插件一加载就闪退，然后下次打开会直接以安全模式加载资源。翻了翻 issue 果然有， [https://github.com/qjebbs/sketch-meaxure/issues/61](https://github.com/qjebbs/sketch-meaxure/issues/61)

不过...有兴趣的程序员们是时候可以考虑帮个忙了。

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/12.png)

以及结合目前看论坛反馈，基本上只有从官网下载的正式版本才能正常使用，破解版目前可能都会有点问题。例如自己从某网站上面找了个同为 `Sketch_70.3` 版本，打开就会出现如下的报错

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/13.png)

**PhotoShop**

早在 2020 年的  11.17 Adobe PhotoShop 就发布了 Beta 版本，但是现在仍没有正式版出来。目前看 Beta 还有不少小问题：[https://forums.macrumors.com/threads/photoshop-for-mac-arm-is-here-beta.2268968/?post=29259426](https://forums.macrumors.com/threads/photoshop-for-mac-arm-is-here-beta.2268968/?post=29259426)。建议有需要使用到的小伙伴还是跑跑兼容版再等等

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/14.png)

### 3. XCode

本想试试我厂的客户端产品，但由于依赖包问题，在 M1 上没能跑成功，后续有空再研究...

### 三.  iOS 应用运行

目前，对于在 app store 上面的应用，你可以通过个人账户的个人中心，选择 `iPhone & iPad Apps`，就能看到你之前下载的 iOS 应用并运行。

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/15.png)

在 app 大行其道的时代，比如有时候想看点东西点个外卖之类的，不想拿手机就会相对方便一些了，对于开发者也不用说一定得去适配 web 端（比如微信小程序在桌面端打开的方式），虽然小了一点，但是很多作为工具类更期望的要求仅仅是能用。

验证了十来个小应用，基本都能打开，当然，iOS 的 app 使用体验跟在手机上还是有不少差异的，比如手势左右滑动，在电脑上你要靠着鼠标去拖拽就相对比较难受，还有有的应用可能有双指缩放等操作。

下图为在 mac 上运行微信读书，翻书似乎适配了支持键盘左右翻页，舒服很多：

![](https://manyu-blog.oss-cn-shenzhen.aliyuncs.com/20210201/16.png)

当你下载完 iOS 的应用之后，图标会出现在你的应用文件夹里面，跟 mac 自带原生的应用混在一起，表面看着并没有任何区别。

对于未验证的应用，可以通过 `.ipa` 文件的方式运行，如有特殊需求可自行搜索下教程，此处未进行系统性的测试。

## 总结

总的来说，距离 M1 发布已经过了好，很多主流软件基本都已经适配或者适配中，目前个人觉得比较硬伤的可能是 Docker，还有看到有评测写到  Android Studio 可以安装编码，但是模拟器无法使用，也会相对比较蛋疼。

视频编辑类的因为个人不好这口，目前没有进行测试，不过基本上 Adobe 全家桶都是在适配中了，AE 啊 Lightroom 都有原生支持。

不过还有一个问题，通常来说官方的兼容出来了，但是对于很多小伙伴可能并没有说所有都是正版可能就需要自己稍微注意下了。

如果恰好有换电脑的打算，或者觉得之前的电脑比较卡什么的，建议可以下手，略香。
