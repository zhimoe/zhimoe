---
title: "如何实现一个拼写检查器[翻译]"
date: "2020-10-25T20:01:27+08:00"
toc: true
categories:
 - "翻译"
tags:
 - code
 - python
---

谷歌AI负责人norvig在07年写的如何实现一个拼写纠正器的经典博文[How to Write a Spelling Corrector](https://norvig.com/spell-correct.html).
上面的链接已经是16年更新过了,程序也更新到了python3.
中文版的翻译 [如何实现一个拼写纠正器](https://blog.csdn.net/suixinsuiyuan33/article/details/69215082) 还是基于07年版本的.

<!--more-->

博文最有意思的地方是大牛记录了如何在飞机上面没有网络的条件下徒手写一个准确率超过70%的拼写纠正器.

