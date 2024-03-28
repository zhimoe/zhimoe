+++
title = 'Rust Error Handling Notes'
date = '2023-07-22T21:12:50+08:00'
categories = ['编程']
tags = ['rust','notes']
toc = true
+++

初学 rust 的时候，上手写代码总是遇到很多不一样的 rust 的`Result`类型，不同 crate 中的函数返回的`Result<T, E>`的`E`都不一样，刚开始都是`unwrap`或者`expect`来处理。如果使用`try!`或者`?`的话总是编译不通过，还是对 Error 转换和处理不熟练。

<!--more-->

### Error 转换
在一个方法中，调用不同的函数会返回不同的 error 类型，需要你将这些类型转换成统一的自定义 error 类型再返回。你有以下几种途径
### 使用 map_err
```rust
fn cook_pasta() -> Result<Pasta, CookingError> {
    let water = boil_water().map_err(|_| CookingError::BoilWaterError)?;
    let pasta = add_pasta(&water).map_err(|_| CookingError::AddPastaError)?;
    Ok(pasta)
}
// 通过 map_err 将 boil_water() 和 add_pasta(&water) 返回的 error 都转换成了 CookingError 类型
```
#### 使用 std::error::Error+From trait
定义自己的 Error 类型并实现 From trait。From trait 用于将 boil_water() 和 add_pasta(&water) 的 error 转换成自定义的 Error。其实就是将`map_err`的逻辑移动到 From trait 中实现，使得方法调用处看起来更简洁。
```rust
pub enum CookingError{
    BoilWaterError(String),
    AddPastaError
}
impl std::error::Error for CookingError{
    // ...
}
impl Display for CookingError{
    // ...
}
// 假设 boil_water 返回的 error 是 NoWaterError
impl From<NoWaterError> for CookingError {
 fn from(s: NoWaterError) -> Self {
        CookingError::BoilWaterError(s)
    }
}
// 假设 add_pasta 返回的 error 是 IoError
impl From<IoError> for CustomError {
    fn from(s: std::io::Error) -> Self {
        CookingError::AddPastaError(s)
    }
}
// 无需 map_err
fn cook_pasta() -> Result<Pasta, CookingError> {
    let water = boil_water()?; // 如果抛出 NoWaterError，自动转成 CookingError::BoilWaterError，下面同理
    let pasta = add_pasta(&water)?;
    Ok(pasta)
}
```


### thiserror
thiserror 可以看作是定义 Error 的一个工具，它只帮你生成一些定义 Error 的代码，别的什么都不做，相当纯粹。如果你在开发一个 crate，那么建议使用 thiserror。

```rust
fn render() -> Result<String, std::io::Error> {
  let file = std::env::var("MARKDOWN")?;
  let source = read_to_string(file)?;
  Ok(source)
}
```
上面的代码无法通过编译，因为`env::var()` 返回的是 `std::env::VarError`，而 `read_to_string()` 返回的是 `std::io::Error`。
为了满足 render 函数的签名，我们就需要将 env::VarError 和 io::Error 归一化为同一种错误类型。要实现这个目的有三种方式：

- 使用特征对象 `Box<dyn Error>`。实际上就是错误类型泛化，失去了具体错误类型的信息，类似于在 Java 中使用`Object`类型。
- 自定义错误类型。比较繁琐，上面的例子在自定义 Error 类型后，需要分别为`env::VarError` 和 `io::Error`实现`From` trait 才行。
- 使用 `thiserror`。简化自定义错误类型的繁琐：

```rust
use std::fs::read_to_string;

fn main() -> Result<(), MyError> {
  let html = render()?;
  println!("{}", html);
  Ok(())
}

fn render() -> Result<String, MyError> {
  let file = std::env::var("MARKDOWN")?;
  let source = read_to_string(file)?;
  Ok(source)
}

#[derive(thiserror::Error, Debug)]
enum MyError {
  #[error("Environment variable not found")]
  EnvironmentVariableNotFound(#[from] std::env::VarError),
  #[error(transparent)]
  IOError(#[from] std::io::Error),
}
```
`thiserror`提供`#[from]` `#[error]`等注解简化错误类型自定义工作。
```rust
#[derive(Error)]
{
    // Attributes available to this derive:
    #[backtrace] // 
    #[error]
    #[from]
    #[source]
}
```

`error(transparent)` 表示转发底层 error 的相关信息，不修改 source 和 Display 相关方法。

### anyhow
anyhow 为你定义好了一个 Error 类型，基本可以看作是一个 Box<dyn Error> ，同时还提供了一些如 context 等扩展功能，用起来更加无脑。如果你在开发一个业务 app，建议使用 anyhow 更加方便。
```rust
use anyhow::Result;

fn main() -> Result<()> {
    let html = render()?;
    println!("{}", html);
    Ok(())
}

fn render() -> Result<String> {
    let file = std::env::var("MARKDOWN")?;
    let source = read_to_string(file).with_context(|| format!("read string from {} failed", &file))?
    Ok(source)
}
```
可以看到这里使用的是`Result<String>`,实际上这是`anyhow`的 type alias：`pub type Result<T, E = Error> = core::result::Result<T, E>;`

`anyhow` 还提供了`with_context`给 error 添加信息，看上去和 expect 类似，只不过那是 panic。

对于一个 app 服务，一些核心登录等功能可能也需要自定义 error 类型，这时可以将`anyhow::Error`作为其中一种 error 类型，即 thiserror + anyhow：

```rust
#[derive(Error, Debug)]
pub enum AppError {
    ...

    #[error(transparent)]
    Other(#[from] anyhow::Error),  // source and Display delegate to anyhow::Error
}
```



参考：
[rust by example - 处理多种错误类型](https://rustwiki.org/zh-CN/rust-by-example/error/multiple_error_types.html)
[Option 和 Result 的一些方法](https://zyfjeff.github.io/%E5%8D%9A%E5%AE%A2/doc/rust/rust-error-handle/)
[rust doc Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html)
[简谈 Rust 中的错误处理](https://lotabout.me/2017/rust-error-handling/)
[细说 rust 错误处理](https://baoyachi.github.io/Rust/rust_error_handle.html#%E7%BB%86%E8%AF%B4rust%E9%94%99%E8%AF%AF%E5%A4%84%E7%90%86)
[?在 Result 中的使用](https://doc.rust-lang.org/rust-by-example/std/result/question_mark.html)
[蚂蚁集团 CeresDB 团队 | 关于 Rust 错误处理的思考](https://rustmagazine.github.io/rust_magazine_2021/chapter_2/rust_error_handle.html)