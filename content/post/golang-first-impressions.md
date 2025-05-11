+++
title = 'Golang 编程初体验'
date = '2025-05-05T21:50:27+08:00'
categories = ['编程']
tags = ['code','golang']
toc = true
+++

总结一下最近使用 golang 写了几个简单的工具和项目的初体验。总体来说 golang 在语法设计上面是非常糟糕的，个人非常不喜欢，但是在运行时和标准库上面，确实解决了之前编程语言的很多痛点，所以目前 golang 确实在业务项目蚕食 Java 的地盘。但是！JVM + Kotlin 还是目前最好的业务技术栈，如果不需要计算性能的话，有了 typing 提示后 web 上面 Python 也是很好的选择，千万不要信了 golang 那些不需要 ORM/使用标准库就够了/显式处理error才是最佳实践的鬼话。

<!--more-->

## 好的方面
### 工具链非常完善好用
虽然 maven 也有根据模板创建初始化项目的命令，但是那个命令长的根本没人记得住，所以才有 https://start.spring.io/ 这种网站，npm、cargo、uv 这种交互命令行才是人道主义，golang 在命令行方面做的非常好，go mod go tidy 等体验并没有比 cargo 差多少，当然目前场景都非常简单。

### 标准库非常丰富
除了 python，golang 的标准库应该是最丰富的。甚至连 http 库都是自带的，在简单的命令工具场景（devops）场景下，golang 几乎可以取代 python 成为脚本运维工具，这也是很多运维 boy 转到 golang 的条件，毕竟运维环境大概率有 golang 但是不能安装第三方库。

### 交叉编译比较简单
写了一个上传 log 到 s3 的小工具，需要编程成二进制集成到 java 项目中一块部署，开发环境是 windows，部署环境是 linux，golang 一行命令就解决了，使用 rust 没能实现 win 交叉编译到 linux，主要是 windows 需要安装的东西太多了，开发电脑不允许，在个人电脑试了一下也可以。当然使用 python 也能解决，但是体积非常大，超过了 5M 允许的限制。

### 编译非常快
得益于 golang 的语法简单，没有 meta programming, 所以编译非常快，可以说启动一个 golang 项目和启动 python 项目几乎一样快，这在 Java 项目上都是难以想象的，大部分使用 Java boy 都需要使用破解版的 JRebel 热更新解决启动慢的问题。

### 启动快，初始内存小
这个其实对于企业 web 开发不算什么太大问题，但是对于个人项目，还是很大的优势。

## 缺点
项目写到一半我就感觉有点恶心了，可能是用过 swift 和 scala 的原因，golang 的语法感觉就是半成品，连 Java 都比他严谨一点。
### 错误处理
golang 的错误处理有两个问题，一是繁琐，整个代码充斥的 if err != nil {}，美其名曰在错误发生的地方处理错误，但是又允许通过 r, _=fn() 省略 err，这个本质上是奖励偷懒者。二是 error 是值，导致传递的信息非常有限，底层 fmt.Errorf + %w 其实是一个 error string 错误链，然后使用 error.Is 判断处理，对于 java 过来的人，没有 stack trace 而 err 只是一个 string 真的非常不习惯，感觉上了生产仅靠一行 error string 定位错误非常困难吧，虽然我还没遇到生产问题。

### 空值设计
这个问题其实比错误处理还搞笑，感觉 golang 的 json 序列化就是玩的，也不知道 golang web 开发前后端是怎么约定空值的，前端还需要额外约定的 json 明显属于 json 包设计有 bug 就这么用了十几年。当然 1.24 的 omitzero 可以解决一部分问题。[JSON 包新提案：用“omitzero”解决编码中的空值困局](https://tonybai.com/2024/09/12/solve-the-empty-value-dilemma-in-json-encoding-with-omitzero/)


### 其他
time 格式、for-range 这些就不吐槽了，官方也在不停修复，但是感觉 golang 和 golang 那些项目都是发版跟玩似的（说的就是 hugo 你），严肃语言怎么也不会把 omitepmty 不支持 time 这种东西发布出来吧，把工作甩给前端开发么？

[What “sucks” about Golang? : r/golang](https://www.reddit.com/r/golang/comments/11o2yfd/what_sucks_about_golang/)
[Go 错误陷阱 --- Go Traps](https://go-traps.appspot.com/#watchman)
[GoLangBooks/50 Shades of Go Traps GotchasandCommonMistakesforNewGolangDevs.pdf at master · diptomondal007/GoLangBooks](https://github.com/diptomondal007/GoLangBooks/blob/master/50%20Shades%20of%20Go%20Traps%20GotchasandCommonMistakesforNewGolangDevs.pdf)
[golang50shad.es](https://golang50shad.es/)
[Golang Sucks - Home](http://www.golang.sucks/)
[I don't like Golang – Martin Vysny – First Principles Thinking](https://mvysny.github.io/golang-sucks/)
[Golang Sucks | Hacker News --- Golang Sucks | Hacker News](https://news.ycombinator.com/item?id=29122481)
[Discover the Dark Side of Go: Why This Popular Language May Sucks | by Roma Gordeev | Medium](https://medium.com/@roma.gordeev/discover-the-dark-side-of-go-why-this-popular-language-may-sucks-ddd3ab2e0eff)
[ksimka/go-is-not-good: A curated list of articles complaining that go (golang) isn't good enough](https://github.com/ksimka/go-is-not-good)
[Golang is not a good language](https://xetera.dev/article/thoughts-on-go)
[Lies we tell ourselves to keep using Golang](https://fasterthanli.me/articles/lies-we-tell-ourselves-to-keep-using-golang)
[My reflections on Golang - DEV Community](https://dev.to/deepu105/my-reflections-on-golang-38jk)
[Why Go Is Not Good :: Will Yager](https://yager.io/programming/go.html)
