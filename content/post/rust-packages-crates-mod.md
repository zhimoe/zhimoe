---
title: "Rust Packages Crates Mod Notes"
date: "2020-01-12T20:03:33+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - rust
---

## packages, crates and modules  
A Cargo.toml is a package. and must have a package name, like 

```toml
[package]
name = "actix-web"
```

A package contains one or more crates;

A package CAN contain as many binary crates as you’d like, but it must contain at least one crate (either library or binary);
use src/main.rs, will build to package-name binary, or use src/bin/b1.rs,src/bin/b2.rs, wil get 2 binaries: b1,b2.

A package must contain zero or one(0或者1个) library crates, and no more.

by convention, package-name is use - (dash,can be _), but lib_name must use _ (underscores, can not be -);

cargo will auto replace the - with _ in package-name to name the  default library crate(lib.rs in src root). Also you can name it in [lib]:


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
# awc,actix-http... all are packages that contains Cargo.toml and src/lib.rs; 
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






