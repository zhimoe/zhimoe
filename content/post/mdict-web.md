---
title: '基于MDX的web词典'
date: 2020-07-01
toc: true
categories:
 - "项目"
tags: 
  - python
  - rust
--- 
Mdict项目是一个糅合了MDX词典、ES例句搜索和AI模型翻译的多源搜索功能Web词典。特别适合部署在内网中学习使用或者给孩子学习使用。
python版本增加了一个机器学习模型翻译。rust版本也有模型，但是还没来得及加。
<!--more-->

### mdict-py

[mdict-py源码](https://github.com/zhimoe/mdict-py)

Mdict项目是一个糅合了MDX词典、ES例句搜索和AI模型翻译的多源搜索功能Web词典。特别适合部署在内网中学习使用或者给孩子学习使用。

特点：
1. 自动识别中英文选择对应mdx词典，目前英文词典包含牛津8和朗文4，中文词典包含汉语词典3
2. 英文尝试拼写纠错功能，动词时态纠错
3. 如果配置了中文会尝试搜索朗文的例句，模糊搜索，对于有英语基础的同学很有用
4. 如果配置了AI模型，会使用机器学习模型翻译，翻译结果比较粗糙，但是可以参考

![mdict-py](/mdict/mdictpy.png)

### mdict-rs

[mdict-rs源码](https://github.com/zhimoe/mdict-rs)

和python版本相比目前只有基本功能： mdx文件解析，查询。
