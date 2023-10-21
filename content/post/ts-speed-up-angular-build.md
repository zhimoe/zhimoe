+++
title = "使用 speed-measure-webpack-plugin 和 Happypack 优化 webpack 打包速度"
date = "2021-09-12T18:02:10+08:00"
categories = [ "编程",]
tags = [ "code", "JS", "webpack",]
toc = "true"
+++


### 问题

一个 ionic app 本地编译需要 8 分钟，提交到流水线编译耗时需要近 40 分钟，从日志看到 webpack 打包步骤耗时最严重。

### 排查与解决

初步判断是流水线使用的容器 CPU 性能较弱或者存储 mount 性能导致的。找流水线同事支持配置了一个纯内存编译流水线，发现还是很慢。接下来使用 webpack 的插件[speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)监控性能。

<!--more-->

在 webpack.config.js 配置：
```js
// webpack.config.js

// npm i --save-dev speed-measure-webpack-plugin
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
// ...the webpack configuration
const prodConfig = {/*...*/}
module.exports = {
    prod: smp.wrap(prodConfig)
};

```

得到下图左侧的结果，可以看到主要耗时都在 angular 的 PurifyPlugin 上。搜索了一番后找到 HappPack 这个多核执行的插件。
![smp-result-before-and-after-happypack](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/happypack.1p8mivl0t3k0.webp)
配置 happypack，由于主要耗时都在`const PurifyPlugin = require('@angular-devkit/build-optimizer').PurifyPlugin;`插件上，这里只需要针对这一个插件配置 happypack 即可。

```js
// webpack.config.js
// 
const HappyPack = require('happypack');
const os = require('os');//获取 cpu core 数量

//loaders 配置
{
    test: /\.ts$/, 
    use: [
    {
        loader: 'happypack/loader?id=ts', // 在所有 loader 之前加上 happypack/loader,id 是 plugins 中定义的
    },
    {
        loader: process.env.IONIC_CACHE_LOADER
    },
    {
        loader: '@angular-devkit/build-optimizer/webpack-loader',
        options: {
            sourceMap: true
        }
    },
    {
        loader: process.env.IONIC_WEBPACK_LOADER
    }
    ]
}

// 在 plugins 中配置 happypack 插件
plugins: [
    ionicWebpackFactory.getIonicEnvironmentPlugin(),
    ionicWebpackFactory.getCommonChunksPlugin(),
    new ModuleConcatPlugin(),
    new PurifyPlugin(),
    new HappyPack({
        id: 'js',
        threads: os.cpus().length,
        loaders: ['@angular-devkit/build-optimizer/webpack-loader']
    }),
    new HappyPack({
        id: 'ts', // 在 loader 中使用
        threads: os.cpus().length, // 开启操作系统 cpu 的最大核心数
        loaders: ['@angular-devkit/build-optimizer/webpack-loader']
    })
    
]
```

再次本地执行，性能提升巨大。在流水线上测试，两次 build 都在 15 分钟左右。