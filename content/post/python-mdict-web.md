+++
title = "基于 MDX 的 web 词典"
date = 2020-07-01
categories = [ "随想",]
tags = [ "python", "rust",]
toc = "true"
+++

Mdict 项目是一个糅合了 MDX 词典、ES 例句搜索和 AI 模型翻译的多源搜索功能 Web 词典。特别适合部署在内网中学习使用或者给孩子学习使用。
python 版本增加了一个机器学习模型翻译.rust 版本也有模型，但是还没来得及加。

### mdict-py

[mdict-py 源码](https://github.com/zhimoe/mdict-py)

Mdict 项目是一个糅合了 MDX 词典、ES 例句搜索和 AI 模型翻译的多源搜索功能 Web 词典。特别适合部署在内网中学习使用或者给孩子学习使用。

特点：
1. 自动识别中英文选择对应 mdx 词典，目前英文词典包含牛津 8 和朗文 4，中文词典包含汉语词典 3
2. 英文尝试拼写纠错功能，动词时态纠错
3. 如果配置了中文会尝试搜索朗文的例句，模糊搜索，对于有英语基础的同学很有用
4. 如果配置了 AI 模型，会使用机器学习模型翻译，翻译结果比较粗糙，但是可以参考

<!--more-->

![mdict-py](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/mdictpy.5i26sz26qyc0.webp)

### mdict-rs

[mdict-rs 源码](https://github.com/zhimoe/mdict-rs)

和 python 版本相比目前只有基本功能：mdx 文件解析，查询。
