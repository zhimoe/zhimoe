+++
title = 'Rust Error Handling Notes'
date = '2023-07-22T21:12:50+08:00'
categories = ['编程']
tags = ['rust','notes']
+++

初学rust的时候，上手写代码总是遇到很多不一样的rust的`Result`类型，不同crate中的函数返回的`Result<T, E>`的`E`都不一样，刚开始都是unwrap或者expect来处理。

<!--more-->

### std::error::Error转换
定义自己的Error类型并实现From trait
### anyhow
anyhow 为你定义好了一个 Error 类型，基本可以看作是一个 Box<dyn Error> ，同时还提供了一些如 context 等扩展功能，用起来更加无脑。 如果你在开发一个业务app，建议使用anyhow更加方便。
### thiserror
thiserror 可以看作是定义 Error 的一个工具，它只帮你生成一些定义 Error 的代码，别的什么都不做，相当纯粹。如果你在开发一个crate，那么建议使用thiserror。

参考：
[rust by example - 处理多种错误类型](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types.html)
[Option和Result的一些方法](https://zyfjeff.github.io/%E5%8D%9A%E5%AE%A2/doc/rust/rust-error-handle/)
[rust doc Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
[简谈 Rust 中的错误处理](https://lotabout.me/2017/rust-error-handling/)
[细说rust错误处理](https://baoyachi.github.io/Rust/rust_error_handle.html#%E7%BB%86%E8%AF%B4rust%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)
[?在Result中的使用](https://doc.rust-lang.org/rust-by-example/std/result/question_mark.html)
[蚂蚁集团 CeresDB 团队 | 关于 Rust 错误处理的思考](https://rustmagazine.github.io/rust_magazine_2021/chapter_2/rust_error_handle.html)
[错误处理：为什么Rust的错误处理与众不同？](http://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/%E9%99%88%E5%A4%A9%20%C2%B7%20Rust%20%E7%BC%96%E7%A8%8B%E7%AC%AC%E4%B8%80%E8%AF%BE/18%20%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88Rust%E7%9A%84%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86%E4%B8%8E%E4%BC%97%E4%B8%8D%E5%90%8C%EF%BC%9F.md)