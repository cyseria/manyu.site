---
title: npm audit 二三事
date: 2018-07-03
tags: ["npm"]
categories: ["项目经验"]
---


## 前言：从一个 bug 说起

最近碰到一个问题，使用一个框架的时候使用公司内部的镜像装怎么都跑不起来, 然而只要改成官方的镜像就没问题，于是去对比了下 `package.lock` 文件，如图：

<img src="https://eux-public.bj.bcebos.com/2018/07/02/df233513dd3e36bafdd4a15a69415bdc.jpg" alt="" width="1202" height="761" />

很明显可以看出用我厂的镜像装出来的 `nanomatch` 是 `1.2.11` 的版本，而官方装的 `1.2.9`，心里一愣我去还能超前，老早听说 `npm@6` 的一主要特征是安全机制，脑洞了一下难道官方 `npm` 源能判断是不是有问题的包还能自动回滚到稳定版本？

研究了一番，正文开始：

## npm audit 命令

首先看[官方文档](https://medium.com/npm-inc/announcing-npm-6-5d0b1799a905)，`npm@6` 的一大更新是新增了 `npm audit` 命令

> **Note: The npm audit command is available in npm@6. To upgrade, run npm install npm@latest -g.**
>
> The `npm audit` command submits a description of the dependencies configured in your package to your default registry and asks for a report of known vulnerabilities. npm audit checks direct dependencies, devDependencies, bundledDependencies, and optionalDependencies, but does not check peerDependencies.

该命令会在项目中更新或者下载新的依赖包之后会自动运行，如果你在项目中使用了具有已知安全问题的依赖，就收到官方的警告通知。

为了复现，随便找了一个没那么官方的项目 [adminMongo](https://github.com/mrvautin/adminMongo) 并执行 `npm install` 试试，果然

```
found 5 vulnerabilities (2 low, 1 moderate, 1 high, 1 critical)
  run `npm audit fix` to fix them, or `npm audit` for details
```

再执行下 `npm audit` 查看详情，列表很清晰。

<img src="https://eux-public.bj.bcebos.com/2018/07/03/9D8FAEFD-85A5-45C2-8DCF-7B312AB9204D.png" alt="" width="1232" height="944" />

执行 `npm audit fix`，发现并没有能自动的帮我 fix 掉这些错误。

```bash
fixed 0 of 5 vulnerabilities in 1295 scanned packages
  2 package updates for 5 vulns involved breaking changes
  (use `npm audit fix --force` to install breaking changes; or do it by hand)
```

根据提示尝试执行 `npm audit fix --force`，发现他帮我把包自动更新到了推荐版本（`supertest@3.1.0`，`mocha@5.2.0`）。

ps. 直接运行 `--force` 的行为不要学习，对于没能自动修复的问题，说明肯定出现了 `SEMVER WARNING` 之类的警告，这意味着推荐的修复版本存在让代码出问题的可能，主要发生在依赖包更改了 API 或者升级了大版本的情况下（semantic version major change，比如我这个项目的 `supertest` 版本是 1.2.0，而推荐的是 3.1.0）。这时候就需要格外的小心甚至需要改动一些自己的代码了。

另外还有一种情况，就是官方发现了漏洞，但是目前没有可用的补丁。这时候就是个体力劳动了 :)

看看这命令到底帮我们干了啥，简单翻译了一下 [官方文档](https://docs.npmjs.com/cli/audit)

```bash
# 扫描项目漏洞把不安全的依赖项自动更新到兼容性版本
npm audit fix

# 在不修改 node_modules 的情况下执行 audit fix，仍然会更改 pkglock
npm audit fix --package-lock-only

# 跳过更新 devDependencies
npm audit fix --only=prod

# 强制执行 audit fix 安装最新的依赖项（toplevel）
npm audit fix --force

# 单纯的获取 audit fix 会做的事，并以 json 格式输出。
npm audit fix --dry-run --json

# 获取详情
npm audit

# 以 JSON 格式打印报告
npm audit --json
```

至于如何关闭安全检查，可以采用以下方式：
- 安装单个包关闭安全审查: `npm install example-package-name --no-audit`
- 安装所有包关闭安全审查
    - 运行 `npm set audit false`
    - 手动将 `~/.npmrc` 配置文件中的 `audit` 修改为 `false`

## audit 报告

通过查看文档和源码发现，`npm aduit` 主要动作就是在 `npm install` 完成之后把需要检查的包的信息发送给一个官方接口, 再根据返回信息生成一个包含包名称、漏洞严重性、简介, 路径等的报告。处理格式化什么的主要在于 [npm-audit-report](https://www.npmjs.com/package/npm-audit-report) 模块，有兴趣的也可以研究下。

那么报告是根据什么生成的呢，根据 audit 的提示信息我们能发现一个 [node 安全平台](https://nodesecurity.io/)，进去首页就看到一句类似『好消息，2018 年 4 月 10 号，npm 正式入驻我们平台』的通知。看来 audit 的数据来源也大概知道了。

该平台的公布一个漏洞的步骤跟大多安全平台类似：

1. 漏洞被披露 or 反馈到此安全平台。
2. 通知维护者相关信息（如果不是维护者自己汇报的）。
3. 通知使用该依赖的用户，给给予推荐性的解决方案。
4. 修复完成之后发布公开补丁，并申请 [CVE](http://cve.mitre.org/) 。
5. 如果 45 天之后还没有修复，则通知超时并公开漏洞。

## 总结

总的来说，安全的问题不容忽视。
很多时候，大家拿到项目之后直接 `npm install`，只要项目能成功运行基本没有人会去关注装了什么。一大堆错综复杂的相互关联的依赖包，就算开发者有安全意识，也缺乏解决安全漏洞的手段。
此时有个官方平台来帮忙管理收集反馈给出报告给出建议等，是一件很值得称赞的事。

## 最后的最后

想到目前已经有很大一部分同学转移去了 `yarn` 阵营，甚至有一些并不知道为什么就转过去了只是当时很多人推荐使用它。其实主要是在 npm v4 时期安裝超慢，才会让 yarn 崛起，在 v5 的时候其实改善不少了，v6 应该是再次强调想夺回这么一部分用户 😐？有兴趣的可以再自行研究下 yarn 和 npm 的对比， 这里有一个 [npm5.7.1 vs yarn1.5.1 数据上的比较以及比较方法](https//github.com/appleboy/npm-vs-yarn)。

再有其实 npm 现在也有很多功能似乎用的人不是很多，篇幅有限简单提几个自认为还不错的。
1.  `npx` 一个随着 `npm 5.2.0` 发布的命令，会帮你执行依赖包里的二进制文件。比如对于没有全局安装的命令你想执行的话就只能 `./node_modules/.bin/webpack -v`，有 `npx` 之后就可以直接使用 `npx webpack -v`。另外还能单次命令而不需要全局安装，比如 `npx create-react-app my-app`
2. 更改 npm 安装包的目录`npm config set prefix <new_path>`（在没有 sudo 权限的场景很有用）
3. 在开发 npm 模块的时候使用 `npm link`
4. ...

至于前面说的问题解决方案，咨询了官方同学以及翻了下 nanomatch 的 [issue](https://github.com/micromatch/nanomatch/issues/15)，发现是包的问题。
(npm 发布了错误版本 1.11 然后及时回滚到了 1.9 并且删除了 11 的版本, 但是我厂镜像没及时同步版本还是 1.11, 而自己刚好卡这间隙)，果然当晚回到家之后再看了一眼一切又恢复正常，就当一个学习知识的经历。

## 参考资料
- [npm audit 文档](https://docs.npmjs.com/cli/audit)
- [【科普】不了解CVE 就不能算是安全圈的人 来看看官方权威解答](https://zhuanlan.zhihu.com/p/27891196)
