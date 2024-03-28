+++
title = "Rust Packages Crates Mod Notes"
date = "2020-01-12T20:03:33+08:00"
categories = [ "编程",]
tags = [ "code", "rust",]
toc = "true"
+++


初学 rust 对于项目的 package 和 crate 的关系，module 和文件的关系有点理不清。做了一点笔记。

## packages and crates
A Cargo.toml is a package. and must have a package name, defined in `[package]` table:

```toml
[package]
name = "actix-web"
```

<!--more-->

A package contains one or more crates. 
a package can only have 0 or 1 library crate, no more; the entry file is `lib.rs`
A package *can* contain as many binary crates as you’d like. the entry file is `main.rs` or `src/bin/b1.rs` etc.

by convention, package-name is use `-` (dash), but lib_name must use `_` (underscores, can not be dash `-`);

cargo will auto replace the `-` with `_` in package-name to name the  default library crate(lib.rs in src root). you can name it in [lib]:

```toml
# rename the lib crate
[lib]
name = "actix_web"
path = "src/lib.rs"

# also you can rename the binary crate:
# it use [[]], array of table in toml, 
# cuz a package can have many binary crate.
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
## crate and mod
A crate is a compilation unit in Rust. 

Whenever `rustc some_file.rs` is called, some_file.rs is treated as the crate file. 
If some_file.rs has mod declarations in it, then the contents of the module files would be inserted 
in places where mod declarations in the crate file are found, before running the compiler it. 
In other words, modules do not get compiled individually, only crates get compiled.

`mod mod_name {}` defines a mod.

`mod mod_name; ` import a mod. cargo will look for mod_name.rs or mod_name/mod.rs and insert the content to current file.

by default the mod is private; but nested mod is allowed to use any code in super mod;

`self` and `super` is to ref the current mod and parent mod;

```rust
fn main(){
    // absolute root path
    crate::music::popular::play();
              
    // relative path
    music::popular::play();
}
```

## use keyword

the `use` keyword brings path into scope;

注意，rust 中`mod`才是 import，`use`只是简化 path 长度。在同一个 package 内部，必须要`mod module_name;`之后才能使用`use module_name::func`.
如果是`Cargo.toml`中的依赖 crate，无需`mod`也无需`use`（使用 full path）就可以使用 crate 的 item。

对于 function，一般约定是 use 函数名上一级：`use mods::foo::bar;`，而不是直接 use 函数`use mods::foo::bar::func_name;`

```rust
//Providing New Names with the as Keyword
use std::io::Result as IoResult;

//Re-exporting Names with pub use
pub use crate::music::popular;

//Use nested paths or the glob operator
use std::{cmp::Ordering, io};
use std::collections::*;

//by default, use is absolute.
use crate::music::popular;
use std::{self,Write}; // std and Write 

//bring a module into scope with `use` and a relative path need start `self`:
use self::music::popular;
```
in most cases you won't need to use `extern crate` anymore because Cargo informs the compiler about what crates are present. (There are one or two exceptions):[sysroot mod](https://doc.rust-lang.org/edition-guide/rust-2018/path-changes.html#an-exception)

### use foo::bar vs use crate::foo::bar
[use in ](https://doc.rust-lang.org/reference/items/use-declarations.html#use-paths)
```rust
mod my_mod {
    pub mod foo {
        pub mod bar {
            pub fn greet() {
                println!("hello rust");
            }
        }

        pub fn greet_twice() {
            // use super::foo::bar::greet;          // ok 相对路径
            // use self::bar::greet;                // ok 相对路径
            // use my_mod::foo::bar::greet;         // error in 2018+ but ok in 2015 edition.
            use crate::my_mod::foo::bar::greet;     // ok 绝对路径，建议使用
            greet();
            greet();
        }
    }
}

fn main() {
    // use crate::my_mod::foo::bar;   // ok 绝对路径 preferred
    // use self::my_mod::foo::bar;    // ok 相对路径
    use my_mod::foo::bar;             // ok 相对路径，省略了 self, error in 2015 edition: relative paths are not allowed without `self`;
    bar::greet();
    use crate::my_mod::foo;
    foo::greet_twice();
}
```

## split up mod into files
1. the mod can be defined in mod_name.rs or mod_name/mod.rs. and nested mod can be in mod_name/nested_mod.rs.
2. you can ref the nested_mod by use `mod nested_mod;` in mod_name.rs;

## pub in struct and enum
1. the Struct members is all private by default even struct name is pub;
2. the Enum members is all public by default if the name is pub;
