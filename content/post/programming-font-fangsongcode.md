+++
title = '支持中文的等宽编程字体：FangSongCode'
date = '2023-10-28T08:12:20+08:00'
categories = ['随想']
tags = ['font','FangSongCode']
toc = true
+++


个人对于字体有严重的强迫症，在过去十年里（以后应该也会）一直在换各种编程字体，也会使用 fontforge 对字体做一些小修改，甚至在 reddit 上面看的咨询编程字体的图片可以一眼认出。 
对于[编程等宽字体](https://www.programmingfonts.org/)，曾经用过比较久的有 Fira Code，Aurulent，TheSansMono，Go Mono，Source Code Pro，Plex Mono，Drafting Mono。也用过一些收费字体例如 MonoLisa，PragmataPro，Operator Mono，Gintronic 等。目前最喜欢的还是 Fira Code（MonoLisa第二），只是 Fira Code 个性突出，在中英文混排中看着非常显眼。

<!--more-->

而对于中文部分，就更加困难了。目前的黑体和宋体都有各自的问题：黑体字形过大，如果是 0.5 宽度的英文（Ubuntu Mono，PragmataPro，Iosevka 等），中文注释过于显眼；宋体则是横笔太细，好看的编程字体都是非衬线的，搭配不是很协调，在显示器上面宋体也不适合长时间阅读。

去年偶然发现网友推荐的聚珍新仿。这是一个民国设计字体，可以说是宋体、行楷的融合体。笔画粗细介于黑体和宋体之间。从此就一直使用这个字体。在 JetBrains IDE 和 VS Code 中设置 fallback 字体非常简单，但是在其他软件和系统上面设置 fallback 就非常费劲，于是决定合并一个字体。

对于英文部分，虽然 Fira Code 是我最喜欢的字体，但是从协调性来看，仿宋字体属于衬线字体，显然和 IBM Plex Mono 最合适。可惜 Plex Mono 等宽字符缺失较多，最终我选择了 Source Code Pro 并对字符 l,r,4,0,1 做了替换或者修改，使其更加适合正文。

对于中英字符宽度问题，中文字体的宽度一般等于 em-size（假设 1000），即 1000，英文字体的宽度就比较多，主流的是 600，半宽的是 500（Iosevka 或者 PragmataPro）,Fira Code 宽度是 1200/1950（em-size），SF Mono 是 1266/2048（em-size），会更加宽一点。 

目前唯一支持 CJK 的等宽字体 Sarasa Gothic 的方案是将思源黑体和 Iosevka 合并，中文宽度 1000，英文 500，但是由于黑体字形过大过黑，显得英文偏小，而且由于窄宽字体设计比较难，个人感觉 Iosevka 和 PramataPro 美感不足。最终选了 Source Code Pro，宽度 600，中文宽度 1000，这样好处是 5 个英文字符和 3 个汉字等宽，勉强能用。

最终的效果如下：

![FangSongCode1](https://cdn.jsdelivr.net/gh/zhimoe/picx-images-hosting@master/pic/fangsongcode.4t8q5v8g5zq0.webp)
![FangSongCode2](https://cdn.jsdelivr.net/gh/zhimoe/picx-images-hosting@master/pic/fangsongcode.7196xry1lvs0.webp)

