---
title: "Associated Type in Rust"
date: "2020-09-20T20:30:41+08:00"
toc: true
categories:
 - "编程"
tags:
 - rust
---

Associated Type and generic diff in rust

## type outside impl
a `type Foo = Bar` outside is just type alias. most used in generic type.

like:` type Thunk = Box<dyn Fn() + Send + 'static>;`

## type inside impl

`type` in an `impl` defines an associated type. associated type可以理解为一个类型占位符,在trait的方法声明中使用.
```rust
pub trait Iterator {
    type Item; // or type T: Display;

    fn next(&mut self) -> Option<Self::Item>;
}
```

<!--more-->

这里Iterator的Implementors将会指定Item的具体类型.例如：
```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
    }
}
```

## diff in associated type and generic
直接将上面的`Iterator`声明为如下泛型不是更简单么？

```rust
pub trait Iterator<T> { 
    fn next(&mut self) -> Option<T>;
}
//  with generice, you can set default type:
/// trait Generic<T = String>
///    where T: Display,

```
主要的区别就是generic可是有任意多个实现,因为`Add<Foo>`和`Add<Bar>`是两个不同的类型.
而associated type只能有一个实现,因为`Iterator`只有一个类型,所以associated type可以用于限制类型.

## when use 
The quick and dirty answer to when to use generics and when to use associated types is: 
Use generics if it makes sense to have multiple implementations of a trait for a specific type (such as the `From<T>` trait). 
Otherwise, use associated types (like `Iterator` and `Deref`).

假设我们实现一个redis 客户端,那么比较适合使用associated types:

```rust
trait RedisCommand{
    type Response;
    fn receive(&self, message: String) -> Result<Self::Response>;
}

impl RedisCommand for PingCommand {
    type Response = String
    
    fn receive(&self, message: String) -> Result<Self::Response>{
        // -- snip --
    }
}

```

