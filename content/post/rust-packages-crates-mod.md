+++
title = "Rust Packages Crates Mod Notes"
date = "2020-01-12T20:03:33+08:00"
categories = [ "编程",]
tags = [ "code", "rust",]
toc = "true"
+++


初学rust对于项目的package和crate的关系,module和文件的关系有点理不清.做了一点笔记.

## packages, crates and modules  
A Cargo.toml is a package. and must have a package name, like 

```toml
[package]
name = "actix-web"
```

<!--more-->

A package(project) contains one or more crates;

A package CAN contain as many binary crates as you’d like, but it must contain at least one crate (either library or binary);
use src/main.rs, will build to package-name binary, or use src/bin/b1.rs,src/bin/b2.rs, wil get 2 binaries: b1,b2.

A package must contain zero or one(0或者1个) library crates, and no more.

by convention, package-name is use `-` (dash,can be `_`), but lib_name must use `_` (underscores, can not be `-`);

cargo will auto replace the `-` with `_` in package-name to name the  default library crate(lib.rs in src root). Also you can name it in [lib]:


```toml
[lib]
name = "actix_web"
path = "src/lib.rs"


# also you can rename the binary:
# it use [[]], array in toml
[[bin]]
name = "my-cool-binary"
path = "src/my-cool-binary.rs"
```

one package(project) can only have one library crate, when the lib continues to get bigger, you want to split up the lib into multiple packages.
cargo introduce you with workspace.

> A workspace is a set of packages that share the same Cargo.lock and output directory.

here is the actix-web package Cargo.toml file:

```toml
[workspace]
members = [
  ".",
  "awc",
  "actix-http",
  "actix-cors",
  "actix-files",
  "actix-framed",
  "actix-session",
  "actix-identity",
  "actix-multipart",
  "actix-web-actors",
  "actix-web-codegen",
  "test-server",
]
# awc,actix-http... all are packages that contains their own Cargo.toml and src/lib.rs; 
``` 


A crate is a compilation unit in Rust. 

Whenever rustc some_file.rs is called, some_file.rs is treated as the crate file. 
If some_file.rs has mod declarations in it, then the contents of the module files would be inserted 
in places where mod declarations in the crate file are found, before running the compiler it. 
In other words, modules do not get compiled individually, only crates get compiled.

`mod mod_name {}` defines a mod.

`mod mod_name; ` cargo will look for mod_name.rs or mod_name/mod.rs and insert the content to current file.

by default the mod is private; but nested mod is allowed to use any code in super mod;

`self` and `super` is to ref the current mod and super mod;

```rust
fn main(){
// absolute path
crate::music::popular::play();
          
// relative path
music::popular::play();
}
```

the Structs members is all private by default even struct name is pub;

the Enums members is all public by default if the name is pub; 

## use keyword

the `use` keyword brings path(crate mod path) into scope;

```rust
//bring a module into scope with `use` and a relative path need start `self`:
use self::music::popular;
//!!!the self:: is no needed in rust 2018+

//use the absolute path
use crate::music::popular;

//make the path to public
pub use crate::music::popular;

//use
use std::{cmp::Ordering,io};
use std::{self,Write};
```

## split up mod into files
1. the mod can be defined in mod_name.rs or mod_name/mod.rs. and nested mod can be in mod_name/nested_mod.rs.
2. you can ref the nested_mod by use `mod nested_mod;` in mod_name.rs;





## summary

1. 简单粗暴的理解,一个项目 == 一个package, 一个package可以包含多个crate. 
2. crate是Cargo的编译单元,也是Cargo.toml中`[dependencies]`的依赖单元.
3. 一个package只能包含一个lib crate(`src/lib.rs`),但是可以在`src/main.rs`或者`src/bin/*.rs`下面包含任意多个bin crate;
4. 对于复杂项目,可以通过cargo的`[workspace]`管理多个crate,这样可以实现一个`Cargo.toml`管理/构建多个lib crate.

5. mod是rust中代码的组织最小单元. `mod mod_name {}` 是定义一个mod;`mod mod_name; ` 表示将`mod_name.rs`或者`mod_name/mod.rs`中的内容插入到当前文件当前位置,并且插入内容被包含在`mod mod_name`中.
6. crate内部的mod引用使用`self::`开头,引用外部crate则使用`crate::`开头.
