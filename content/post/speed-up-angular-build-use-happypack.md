---
title: "使用speed-measure-webpack-plugin和Happypack优化webpack打包速度"
date: "2021-09-12T18:02:10+08:00"
toc: true 
categories:
 - "编程"
tags:
 - code
 - JS
 - webpack
---

### 问题

一个ionic app本地编译需要8分钟,提交到流水线编译耗时需要近40分钟,从日志看到webpack打包步骤耗时最严重.

### 排查与解决

初步判断是流水线使用的容器CPU性能较弱或者存储mount性能导致的.找流水线同事支持配置了一个纯内存编译流水线,发现还是很慢. 接下来使用webpack的插件[speed-measure-webpack-plugin](https://www.npmjs.com/package/speed-measure-webpack-plugin)监控性能.

<!--more-->

在webpack.config.js配置：
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

得到下图左侧的结果,可以看到主要耗时都在angular的PurifyPlugin上.搜索了一番后找到HappPack这个多核执行的插件.
![smp-result-before-and-after-happypack](https://cdn.staticaly.com/gh/zhimoe/zhimoe.pic@main/pic/happypack.1p8mivl0t3k0.webp)
配置happypack,由于主要耗时都在`const PurifyPlugin = require('@angular-devkit/build-optimizer').PurifyPlugin;`插件上,这里只需要针对这一个插件配置happypack即可.

```js
// webpack.config.js
// 
const HappyPack = require('happypack');
const os = require('os');//获取cpu core数量

//loaders配置
{
    test: /\.ts$/, 
    use: [
    {
        loader: 'happypack/loader?id=ts', // 在所有loader之前加上happypack/loader,id是plugins中定义的
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

// 在plugins中配置happypack插件
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
        id: 'ts', // 在loader中使用
        threads: os.cpus().length, // 开启操作系统cpu的最大核心数
        loaders: ['@angular-devkit/build-optimizer/webpack-loader']
    })
    
]
```

再次本地执行,性能提升巨大.在流水线上测试,两次build都在15分钟左右.