---
title: "HUGO 折腾小记"
date: 2019-07-27 
tags: ["hugo","linux"]
categories: ["随手调研"]
---

# 前言
> 没事折腾一下把尘封已久的 hexo 换成了 hugo，试了一下编译速度明显比 hexo 快很多。怎么使用主题网上介绍比较多，这里记录下其他的东西。

## 部署
首先是部署，无非两种方案：

1. Github pages。优点：免费，快捷。缺点：编译麻烦。
 大多教程都是告诉你本地编译后把 `publish` 推上去。个人觉得这样比较恶心，写博客就专心写博客。而且假如你在手机里浏览时，发现一个错别字，顺手就在 `GitHub` 的 `Web` 界面就把错别字改了，然而这样并不会重新生成静态页面和部署。
因此一种高级点的方案是配合 `travis`，如果稍微洁癖点可以建两个仓库或者两个分支，一个当源码，一个当 `github pages` 读取的 `repo`，每次提交之后自动编译提交，所谓眼不见为净。

2. 自己服务器做编译。
优点：不污染代码库，甚至能直接本地跟服务器文件夹同步，改完编译 github 都不用推送。毕竟，写作应该是一个相对单纯的事情。
缺点：买服务器要钱，linux 不熟的还折腾。

个人选择了方案二，主要是刚好有台阿里云服务器 + 轻度洁癖。

### Hugo 安装
其实参照 [官方介绍](https://gohugo.io/getting-started/installing/)  已经介绍得很完美了，这里补充点注意事项。
#### macOS
 `macOS` 最简单，直接 `homebrew` 大法， 比较恶心的是 `homebrew` 每次装东西前都会自己去 update，墙内的话建议换个源。
#### CentOS
本来在介绍里没看到 CentOS 太好的方案，打算源码编一把，无奈墙太高，选了别的方式。

1. 首先需要 `golang` 的环境
```bash
yum -y install golang
# check version
go version
```

2. Linux 官方建议是使用 `snap` 的方式安装，但是对于很多系统 `snap` 也是没有。作为一个装东西脸黑的人，选择用 `epel` 。

 添加 `epel repo`

```bash
vim /etc/yum.repos.d/hugo.repo 
```

输入以下内容，最好去官方看看 [最新的文件地址 ](https://copr.fedorainfracloud.org/coprs/daftaupe/hugo/) ，这里是 `Hugo 0.55` 版本，主要核对 `baseurl`。

```
[daftaupe-hugo]
name=Copr repo for hugo owned by daftaupe
baseurl=https://copr-be.cloud.fedoraproject.org/results/daftaupe/hugo/epel-7-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://copr-be.cloud.fedoraproject.org/results/daftaupe/hugo/pubkey.gpg
repo_gpgcheck=0
```

安装

```bash
yum -y install Hugo
hugo version
```

一个卡了一会的点是 hugo 编译 `scss` 需要 `extends` 的版本，装了几次感觉还是不大行，但是也没有任何报错，主题编译的时候不生效，最后还是把 `css/js` 这些十年都不会变的样式文件产出直接打了出来。

### Nginx

网上教程很多，不多说。

```bash
yum install epel-release
yum install nginx

whereis nginx
# => nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx

#start
nginx
ps -ef |grep nginx
nginx -s reload
nginx -s stop
```

### Webhook 
也不多说，使用的 [github_webhook_handler](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Frvagg%2Fgithub-webhook-handler)  +  [pm2](https://www.npmjs.com/package/pm2) 

## 主题

一直以来都想着自己搭个博客系统，然后自己撸个主题。后端数据库都耍过一阵，但是最终还是懒，到了很多东西能用就行的心境。不过也一直告诉自己，要有造轮子的能力和不造轮子的决心。

主题使用的  [leaveit](https://themes.gohugo.io/leaveit/) ， fork 下来参考 bear 改了改配色和格式，一些新版的修复，还有文字统计和阅读时间大概推算等。

目前 修改/增加 的东西：

* 修改主题配色和标题样式等细节调整
* 新增进度条
* 新增字数统计功能和阅读时间功能
* 新增侧边栏
* live2d

基本都是用第三方的东西，就懒得详细些了，有兴趣的可以参见 [Github 的提交记录](https://github.com/cyseria/LeaveIt/commits/master)

