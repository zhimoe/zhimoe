+++
title = 'bash 和 dockerfile 的复制命令是否是否需要包含/'
date = '2026-08-09T22:37:32+08:00'
categories = ['编程']
tags = ['code','bash']
toc = true
+++

```cp -r tmp/src tmp/dst```对于 cp 和 mv 命令 总是不知道什么时候带尾部/的区别，以及 dockerfile 中 COPY 和 bash 的 cp 区别。

<!--more-->

1. src 和 src/ 默认无区别，都是复制目录和文件，但是 src 如果是 softlink，src 是 cp 链接本身，src/则会解引用 情况比较复杂 有些 linux 会报错 有些是复制目录里面内容。
2. dst 和 dst/ 如果 dst 自身是目录一定要加/。没有/ 时，且 dst 不存在，`cp` 和 `mv` 都认为是 rename src（无论 src 是文件夹还是文件，存在则报错） ；如果 dst 目录存在，则变成 dst/src/a.txt。

所以 src 建议不加/，dst 如果希望是目录，则必须加上/。

Dockerfile 中则有点不一样，COPY 不会保留 src 目录名

```
COPY src /opt/
```
这里 src 或者 src/都是复制里面的内容，如果希望保留 src，则需要在 dst 参数中写上：`COPY src /opt/src`