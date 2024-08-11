+++
title = '在 CI 环境配置 puppeteer 的注意事项'
date = '2024-08-11T15:21:54+08:00'
categories = ['编程']
tags = ['code','puppeteer']
toc = true
+++

在流水线 CI-CD 环境中打包部署 puppeteer 服务的一些注意事项。

<!--more-->

1. puppeteer 新版推荐使用配置文件（.puppeteerrc.js），而不是 .npmrc 环境变量。配置方式参考：https://pptr.dev/guides/configuration/
   
2. npm mirror 的二进制文件下载地址、web url 地址、npm 包配置地址是不一样的。
   - `https://registry.npmmirror.com/binary.html` 是二进制镜像文件的 WEB URL 用于浏览器查看文件的，这个地址配置无法下载二进制文件。
   - `https://npmmirror.com/mirrors/` 是二进制下载地址，例如下来 puppeteer 包的 chrome-for-testing 依赖，sass 包的 node-sass 的依赖。更多二进制文件地址参考[这里](https://github.com/cnpm/binary-mirror-config/blob/master/package.json)
   - `npm config set registry https://registry.npmmirror.com` 是淘宝的 npm 包地址。

3. 默认 puppeteer 下载的 chrome 文件在 ~/.cache/puppeteer 中，这对于 CD 构建镜像非常不友好，可以通过`cacheDirectory: join(__dirname, '.cache', 'puppeteer'),`修改到当前目录。

[参考项目 puppeteer-cn-config](https://github.com/zhimoe/puppeteer-cn-config)