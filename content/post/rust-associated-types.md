+++
title = "Associated Type in Rust"
date = "2020-09-20T20:30:41+08:00"
categories = [ "编程",]
tags = [ "rust",]
toc = "true"
+++


Associated Type and generic diff in rust

## type outside impl
a `type Foo = Bar` outside is just type alias. most used in generic type.

like:` type Thunk = Box<dyn Fn() + Send + 'static>;`

## type inside impl

`type` in an `impl` defines an associated type. associated type 可以理解为一个类型占位符，在 trait 的方法声明中使用。
```rust
pub trait Iterator {
    type Item; // or type T: Display;

    fn next(&mut self) -> Option<Self::Item>;
}
```

<!--more-->

这里 Iterator 的 Implementors 将会指定 Item 的具体类型。例如：
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
/// trait Iterator<T = String>
///    where T: Display,

```
主要的区别就是 generic 可是有任意多个实现，因为`Iterator<Foo>`和`Iterator<Bar>`是两个不同的类型。
而 associated type 只能有一个实现，因为`Iterator`只有一个类型，所以 associated type 可以用于限制类型。

## when use 
The quick and dirty answer to when to use generics and when to use associated types is: 
Use generics if it makes sense to have multiple implementations of a trait for a specific type (such as the `From<T>` trait). 
Otherwise, use associated types (like `Iterator` and `Deref`).

假设我们实现一个 redis 客户端，那么比较适合使用 associated types:

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

