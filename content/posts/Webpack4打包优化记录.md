---
title: "Webpack4 打包优化记录"
date: 2019-10-07
tags: ["webpack", "优化"]
categories: ["工程化"]
---

> 随着项目的生长，项目的打包速度&启动速度也变得越来越慢，于是决定开始一系列的优化之旅。首先最简单的，从 webpack 入口，在不修改代码的情况下，看看我们能做什么？

## 一、项目简介
本次尝试的项目为一个多入口应用，共 10 个模块，目前打包出来的目录结构大致为：
```bash
# 公共依赖
├── framework
│   ├── framework.99f5b067.bundle.js        129kb
└── vendor
    ├── vendor.d6cea408.bundle.js           2.2MB

# 组件模块（共10个）
├── index                                     
│   ├── index.0807753e.bundle.js              
│   └── index.d88a14292e704db0d999.css        
├── index.html
```

整体加起来项目大小打出来为 `4.3MB`，时间初步估计一分钟左右。

## 二、打包体量

很明显我们可以看到，`vendor` 相对其他文件，已经稍微有点大了，因此我们首先对打包产物的体量进行一定的优化。

瞅瞅当前项目的  `optimization` 配置，看着好像似乎没什么太大的问题：

```js
optimization: {
         splitChunks: {
             chunks: 'all',
            minSize: 30000,
            maxSize: 0,
            minChunks: 1,
             cacheGroups: {
                framework: {
                    test: 'framework',
                    name: 'framework',
                    enforce: true,
                 },
                 vendors: {
                     priority: -10,
                     test: /node_modules/,
                     name: 'vendor',
                     enforce: true,
                 },
             },
         },
}

```

然而再去仔细查看 [webpack 默认的配置](https://www.webpackjs.com/plugins/split-chunks-plugin/#optimization-splitchunks)，可以发现，有部分配置其实已经是默认的，并没有太大的意义，比如整个  `vendors`，以及 `minSize`，`minChunks` 等，至于 maxSize 并没找到太好的出处，直接干掉试试。

再有 framework 中的 test 为匹配文件，实际上经检查发现这里为入口的配置即只是对 `react`，`react-dom` 这两个做了规整，但是对于公用的其他内容都没有很好的去合并。

```js
entry: {
     ...entries,
     framework: ['react', 'react-dom'],
},
```

于是我们删除没太大影响的默认配置，再对公共模块做一定的抽取

```js
optimization: {
         splitChunks: {
             chunks: 'all',
             cacheGroups: {
                commons: {
                    // 对于引用次数超过 2 次的放去 common
                    minChunks: 2,
                    maxInitialRequests: 5,
                    minSize: 0,
                    name: 'commons'
                },
             },
         },
}
```

这么操作之后，平均每个模块打包出来的体量都有一定的下降，整体打包的大小为 3.7MB。一下为公共部分的大小，发现 `common` 这个文件变得特别大

```
├── common
│   └── common.js                           2MB
│   └── common.css                          265KB
├── framework                               2kb
└── vendor
    ├── vendor~a                            96kb
    ├── vendor~b.                           368kb
```

这个时候我们该拿出分析工具了，比如非常常见的  [BundleAnalyzerPlugin](https://www.npmjs.com/package/webpack-bundle-analyzer)，启动一下运行看看

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/24c80493f7b297f7862c4b8740542b2d.png)


有个 `emoji-picker` 竟然有 `1M` 多，经查看每个表情为 `9kb`，这部分资源都打包成了 `base64` 也就是放在了 `js` 里面。考虑到我们的资源实际上是会部署在 `bos` 上，有 http2 的多路复用加成，直接打包出没有太大问题。修改 loader 配置：

```
 {
     test: /\.(jpe?g|bmp|png|gif)$/,
     loader: 'url-loader',
     options: {
         limit: 5120, // 限制 5kb，原来为 10240，即 10kb
         outputPath: 'resourse',
     },
},

```

优化后：总大小为 3.1MB，资源文件夹从原来的 456kb 变成了 1.3MB，common.js 则从 2MB 下降为 872kb。

再继续看，`lottie` 这货大概 `550kb` 左右，其实我们用它主要是个 `loading` 动画，记一个后续代码优化事项。然后本次还是从纯配置优化角度入手，因此继续分包。另外还有 swiper 也可以抽出来。

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/38b3779bf54506652c29af964336ecde.png)


优化后的配置：

```js
 splitChunks: {
            chunks: 'all',
            cacheGroups: {
                commons: {
                    minChunks: 2,
                    maxInitialRequests: 5, // 最大初始化请求书，默认1
                    minSize: 0,
                    name: 'commons',
                },
                lottie: {
                    test: /node_modules\/lottie/,
                    name: 'commons/lottie',
                    priority: 1,
                },
                swiper: {
                    test: /node_modules\/swiper/,
                    name: 'commons/swiper',
                    priority: 1,
                },
            },
        },
```

打包产物，整体大小变为 3.1MB，发现原来生成的 `vendor` 也都没了

```
├── common
│   └── common.js                           610kb
│   └── common.css                          265kb
│   └── lottie.js                           262kb
│   └── swiper.js                           138kb
│   └── swiper.css                          14kb
```

对应的，每个入口模块都已经变得非常小，整体打包后的大小也从 4.3MB -> 3.1MB，大小优化暂时到这里为止。

```bash
#  du -d 1 -h | sort -h
...
 36K	./interest
 52K	./subject
128K	./article
300K	./index
1.2M	./commons
1.6M	./resourse
```

## 三、打包速度

接下来看看打包速度，影响开发体验的一个重要环节。首先对于打包速度怎么衡量其实就是一个问题，比起手动打 `log` 的形式，这里我们直接使用[speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)，能更精确的分析出每个步骤的耗时。

以下为关了 `sourcemap` 在生产环境的打包速度：

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/59ba21b8353e0fb01c971d0526b3c7e9.png)


可以看到，主要的耗时在于，`modules with no loaders`，还有 `sass-loader` 上面，`babel-loader` 也相对有点慢。

首先，我们要想想，为什么慢，这又是一道类似 从 url 输入到页面渲染发生了什么的题目。打包过程中，常见影响构建速度的地方有哪些？就得从 webpack 打包原理来看了，个人认为：

1. 查找。开始打包，需要获取所有的依赖模块，因此需要搜索所有的依赖项，也就是对应我们配置中的 `include`, `exclude` 之类的，范围越小他需要查找的地方就越少

2. 解析。查找完模块之后，webpack 开始去做对应的解析处理，例如 scss，typescript 之类的转换，这一系列肯定也是随着文件的增多时间也会跟着增加。

3. 打包。解析完毕之后，对于 webpack，他的每个文件都是模块，但是实际我们运行的时候，肯定会进行一定的代码合并，即将所有解析完成的代码，打包到一个文件中。在生产环境下，代码合并完毕之后还需要进行压缩处理，通常这也是一个常见的耗时操作

然后我们一个个的看，参照 web pack 的 [build-performance]([Build Performance | webpack](https://webpack.js.org/guides/build-performance/))。开始实验

### 并行

对于解决所谓的性能问题一直都有一个梗：性能问题最快的解决方案就是加机器。对于本地打包同理，我们可以想想怎么快速的尽可能的榨干机器的性能。
由于 `nodejs` 为单线程，因此可以从如何将编译并行开始。

曾经是有 `happypack` 的工具，但是对于 `webpack4` 已经做了很多优化，包括作者也是建议使用其自身的 [thread-loader](https://www.webpackjs.com/loaders/thread-loader/)，对应原文  [HappyPack Notes](https://github.com/amireh/happypack#happypack--)。

由于每个 worker 都是一个单独的有 600ms 限制的 node.js 进程，同时跨进程的数据交换也会被限制。建议耗时的 loader 上使用，因此计划对 babel-loader 和 scss 等样式处理相关的 loader 添加试试。

先对 `babel-loader` 该加上配置，打包每个模块之间没有太大的进步，不过整体速度稍微快了一点点

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/dad3443a2e0b2d84c503a0febd2ce284.png)


然而当尝试对 `scss-loader` 加上 `thread-loader` 之后，发现事情跟想象中的就不大一样了，满屏的红色：`The "path" argument must be of type string. Received type undefined`。

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/d413912704ac85bbfd47225aa6d358f2.png)


经过一系列排查，在 `sass-loader` 的文件中找到了报错位置，打 `log` 发现的确 `rootContext` 是为空的，但是这个是在 `sourceMap` 开启的时候才会进入的：

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/8f8866c585bfdacae75b6f60b7fbfa5c.png)


又经过一系列 `debug`，发现主要原因在于 `TerserPlugin`，其有个配置为 `sourceMap: true`，经查阅发现他只有在 `devtools` 设置了 `sourcemap` 之后才会正常运行，参考 [TerserWebpackPlugin | webpack](https://webpack.js.org/plugins/terser-webpack-plugin/#sourcemap)。

因此，我们只需要对  `TerserPlugin`  的 `sourceMap` 关闭，或者对于打包构建的时候加上 `devtool: ‘source-map’`就能正常运行。我们将配置优化一下，顺手加上并行缓存之类的配置：

```js
 new TerserPlugin({
      cache: true,    // 缓存
      parallel: true, // 并行
      exclude: /\.min\.js$/,
}),
```

另外，样式相关的 loader 顺序也很重要：

```
styleLoader,
'thread-loader',
cssModuleLoader,
postcssLoader,
'resolve-url-loader',
sassLoader,
```

TODO：然而...
![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/fd3827c4869fe03d4aef44164aa4f3d6.png)

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/c9d5c21cfa16067fccf74808ef2e56aa.png?x-oss-process=image/resize,w_200)

#### 注意事项

我们对 `babel-loader`  和 `sass-loader` 加入该配置，并配合使用预热 `worker` 功能。

```js
const threadLoader = require('thread-loader');

threadLoader.warmup({
  // pool options, like passed to loader options
}, [
  'babel-loader',
  'sass-loader',
]);
```

TODO：
但是实际操作中发现，加上这个预热之后，build 进程就杀不掉了..


### 缓存

说到缓存，一般会想到利用空间去换取时间，对于 webpack 常见的 dll，社区上也都诞生了很多伟大的，励志于缩短 Webpack 构建时间以及减小成本的解决方案，包括但不限于：

* cache-loader
* DllReferencePlugin
* auto-dll-plugin
* thread-loader
* happypack
* hard-source-webpack-plugin

对于 `webpack4`，`vue-cli` 和 `create-react-app` 并没有使用到 dll 技术，而是使用了更好的代替着：[hard-source-webpack-plugin](https://github.com/mzgoddard/hard-source-webpack-plugin)。参照 vue-cli 的 [issue](https://link.zhihu.com/?target=https%3A//github.com/vuejs/vue-cli/issues/1205) ，以及 create-react-app 中对于 [pr](https://link.zhihu.com/?target=https%3A//github.com/facebook/create-react-app/pull/2710) 拒绝的解释。

#### hard-source-webpack-plugin

我们这里先使用大杀器

```js
// webpack.config.js
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');

module.exports = {
  plugins: [
    new HardSourceWebpackPlugin()
  ]
}
```

TODO：
小插曲：很开心的添加完毕，二次打包的时候发现读取的缓存并不成功

![](http://manyu-blog.oss-cn-shenzhen.aliyuncs.com/37b8d3b6f4a6219ec08056933d0381c3.png)


#### babel-loader cache

```js
{
    test: /\.(js|jsx|ts|tsx)$/,
    use: ['thread-loader', 'babel-loader?cacheDirectory'],
    exclude: /node_modules/,
},
```

 babel-loader cacheDirectory，配置之后多执行几次，会发现速度快了很多，`babel-loader` 只需要 8s 了。


## 四、其他

一些有应该一定效果，但是没有实际去测试或者对本项目影响不大的

* 对于耗时的 loader，使用 `include`，尝试对 babel-loader 和 scss-loader 进行了处理。发现并没有明显提升。

* resolve。配置 webpack 如何寻找模块所对应的文件，也是尽可能的缩小范围。项目中已经比较全面的配置了 `extensions` 和 `alias` 两个属性，没再做试验。

* [noParse](https://webpack.docschina.org/configuration/module/#modulenoparse)。如果一些第三方模块没有 `AMD/CommonJS` 规范版本（当然最好还是能找到对应的模块化版本），可以使用 `noParse` 来标识这个模块，这样 Webpack 会引入这些模块，但是不进行转化和解析。例如：jquery。

* [ignore-plugin](https://webpack.js.org/plugins/ignore-plugin/)。webpack 的内置插件，忽略第三方包指定目录。例如: moment (2.24.0版本) 会将所有本地化内容和核心功能一起打包，我们就可以使用 `IgnorePlugin` 在打包时忽略本地化内容。

* [externals](https://webpack.js.org/configuration/externals/)。对于通用外部依赖，我们可以将这些存储在 `CDN` 上(减少 `Webpack` 打包出来的 `js` 体积)。在 `index.html` 中通过 `<script>` 标签引入。对于 http2 不考虑复用的情况下其实挺多包都可以玩玩试试。


## 五、结尾
以上，是作为 webpack 配置工程师的一点简单挣扎，以 Webpack 官网的一句话结尾。

> **Don’t sacrifice the quality of your application for small performance gains!***Keep in mind that optimization quality is, in most cases, more important than build performance.*

相关质量不错的文章：

* [从构建进程间缓存设计 谈 Webpack5 优化和工作原理](https://zhuanlan.zhihu.com/p/110995118)

* [五种可视化方案分析 webpack 打包性能瓶颈](https://juejin.im/post/6844904056985485320)

* [cache-loader vs hard-source-webpack-plugin](https://github.com/webpack-contrib/cache-loader/issues/11#issuecomment-328480598)
