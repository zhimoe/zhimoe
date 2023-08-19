+++
title = "最佳编程字体"
date = "2020-08-21T22:02:50+08:00"
categories = [ "随想",]
tags = [ "font", "aurulent", "firacode",]
toc = "true"
+++


个人对于编程字体有一点点洁癖，在尝试十几个字体后，终于使用FontForge和fontline.py动手修改制作自己的编程字体:Aurulent和 Fira Code。

### Aurulent
[字体下载](https://github.com/zhimoe/programming-fonts)

1. 小写字母来自Aurulent Sans Mono，其他基于Fira Code.Aurulent Sans Mono风格和SourceCodePro非常像，胖宽型，大开大合，简单却有具有设计，特别是字符g，a，p，y，s。
2. 字母r的思路来自[gintronic](https://www.programmingfonts.org/#gintronic). 优点是在低分辨率屏，r的末尾不会被hint只剩下尖尖。
3. 问号？也来自gintronic字体，非常漂亮。

<!--more-->
Aurulent效果
![Aurulent](https://cdn.staticaly.com/gh/zhimoe/zhimoe.pic@main/pic/aurulent.4sz6bmooqf80.webp)

### Fira Code
Fira Code是全网最受欢迎的字体，但是这个r实在过于fancy，所以重新构建了一个版本，只提供regular和bold两个字重

### MonoLisa
目前在用也是最喜欢的收费字体， [MonoLisa](https://www.monolisa.dev/)完美体现了官网上面的"font follows function"宣言，整个字体在代码阅读中有一种从左到右的流动感。也是自己第二次付费字体(第一次是方正)
唯一不足的是和fira code相比，开发者在hint方面不太行，每个版本总是会有一些hint问题，在非4K显示器上面效果会很糟糕，例如"=>"的等号会明显上下粗细不一致。因为自己都是4K显示器，干脆就使用fontforge进行dehint处理。

### 其他编程字体
个人比较喜欢的字体有

1. [SourceCodePro](https://github.com/adobe-fonts/source-code-pro)， 只是这个r在低分辨率下一塌糊涂， 结合office code pro做了一个更适合正文的[SourceCodePro版本](https://github.com/zhimoe/programming-fonts/blob/master/screenshots/scp.png)
2. [Adobe LetterGothic](https://fonts.adobe.com/fonts/letter-gothic)： 这是一个经典的IBM打字机字体。这个字体经典在于字符r是我认为所有字体里面设计的最漂亮的， 在字体设计中，感觉r是最难设计的，像fira code这种r， 有点过于fancy，很容易吸引你的目光; 像source code pro那种超级简洁， 在win下面渲染除非是高分屏，否则一塌糊涂.除了字符r，  letter gothic作为1960时代打字机默认字体之一，在字符n，u的角上，都保留了非常漂亮而含蓄的细节，这一点，我非常吐槽jetbrains mono字体，居然把小写u的尾巴去掉，声称可以加快阅读， 或许能提速，但是丢了美感. LetterGothic具体的效果看我[之前的推文Thread](https://twitter.com/_zhimoe/status/1422032997730058241?s=20).
3. [TheSansMono](http://www.lucasfonts.com/fonts/the-sans/info): 经典等宽字体。你可以在很多书上面看到这个字体.斜体是所有字体最好看的，收费，作者也是windows经典的代码字体Consolas作者。

Letter Gothic效果   
![Adobe LetterGothic](https://cdn.staticaly.com/gh/zhimoe/zhimoe.pic@main/pic/letter-gothic.5krkimcvicw0.webp)
  

### 非编程英文字体推荐
#### Sans类
一般黑体中文适合搭配Sans英文字体，推荐经典的Open Sans，Inter，Lato，Fira Sans。Inter风头正盛，很多网站包括2023的jetbrains IDE UI字体都换成这个了，特点就是没有任何特色。个人认为Lato设计细节最佳，但确实不适合用于UI，但是在网站正文中使用非常不错。
收费字体中`Sana Sans`在正文中效果也不错，播客网站changelog用的就是这个字体。

#### Serif类
一般宋体中文适合搭配衬线英文字体，推荐Palatino和Merriweather，后者是google font上面排名第一的serif字体，缺点是字体偏粗。

#### 手写体
大部分手写体都不适合阅读，但是`Alegreya Sans`斜体比较合适，在手写风格和可读性达到较好平衡。