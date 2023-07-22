+++
title = 'Rust Error Handling Notes'
date = '2023-07-22T21:12:50+08:00'
categories = ['编程']
tags = ['rust','notes']
+++

初学rust的时候，上手写代码总是遇到很多不一样的rust的`Result`类型，不同crate中的函数返回的`Result<T, E>`的`E`都不一样，刚开始都是unwrap或者expect来处理。

<!--more-->

### std::error::Error转换
    

参考：
[Option和Result的一些方法](https://zyfjeff.github.io/%E5%8D%9A%E5%AE%A2/doc/rust/rust-error-handle/)
[rust doc Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
[简谈 Rust 中的错误处理](https://lotabout.me/2017/rust-error-handling/)
[细说rust错误处理](https://baoyachi.github.io/Rust/rust_error_handle.html#%E7%BB%86%E8%AF%B4rust%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)