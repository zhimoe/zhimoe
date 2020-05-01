---
title: "Rust Ownerships Lifetimes教程"
date: "2020-02-22T15:55:13+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - rust
---

some notes on rust ownership,reference,string and &str, and lifetimes
<!--more-->


### rust ownership
```rust
//heap and stack: stack is store data that known,fixed size.
//Keeping track of what parts of code are using what data on the heap, minimizing
//the amount of duplicate data on the heap, and cleaning up unused data on the heap
//so you don’t run out of space are all problems that ownership addresses.

//ownership rules:
//Each value in Rust has a variable that’s called its owner.
//There can only be one owner at a time.
//When the owner goes out of scope, the value will be dropped.

// stack only data assignment(栈上数据) will make a copy operation, since it is fixed size, the copy is fast
// use h.clone() make a heap data deeply copy.
// impl the Copy trait can make a type still usable after assignment
// Copy trait can not use with Drop trait,Drop可以理解为destructor,当数据超过自己的scope时,drop()方法被调用;
fn copy() {
    let x = 5;
    let y = x; //copy the value(5) in the stack,since it is fixed-size, the copy operation is fast

    let s1 = String::from("hello"); //String 和 &str区别见后文
    let s2 = s1; //s1 is invalid
    // println!("{}, world!", s1); //error, the "hello" ownership move to s2

    let s3 = s2.clone(); //copy the heap value("hello"), impl the Clone trait
    println!("{}, world!", s2); // s2 still usable
}

// passing function arguments or return value by function is same as 
// assigning a value to a variable, you need take care the ownership of heap value,
fn ownership() {
    let x = 5;
    let x10 = plus10(x);// x still usable since the x is stack data
    println!("{}", x);
    println!("{}", x10);

    let s = String::from("hello");
    takes_ownership(s); //s's value moves into the function and so is no longer valid here
    //println!(s) ;//error!
}

fn plus10(i: i32) -> i32 { // since the i is primitive in stack, so the function return a new value  
    i + 10 
}

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop()` is called. The backing memory is freed.
```
推荐阅读[A closer look at Ownership in Rust](https://blog.thoughtram.io/ownership-in-rust/)

### References and Borrowing:
```rust
// since the ownership is too hard to track by coder's eye, rust introduce the ref and borrowing
// a function that accept a ref will not takeover a value's ownership when the function is called
// also will not drop the value's backend memory when function is return.


// a variable can only have one mut ref or many immutable ref in a same scope;

//dangling reference
fn dangle() -> &String {
    let s = String::from("dangle ref");
    &s //error
}// the s is dropped, but the function try to return s reference


### String vs str vs &String vs &str
//1. String is heap string buffer
//2. &String is a ref of String
//3. str is unknown immutable sequence of utf8 bytes stored somewhere in memory. the memory may be:
//  3a. in binary: a string literal "foo" is a &'static str. The data is hardcoded into the executable and loaded into memory when the program runs.
//  3b. in heap: Strings implement Deref<Target=str>, and so inherit all of str's methods.
//  3c. in stack: when use str::from_utf8(x).unwrap(); x is stack-value ref

//> the &str param can accept a &String since the String implement Deref<Target=str>.
// 即接受&str的地方都可以使用&String

//!!! since the str is unknown size, one can only use it by &str, called slice. slice is a view of some data. 


fn str() {
    let s = "hello str";//The type of s here is &str: it’s a slice pointing to that specific point of the binary.
    // This is also why string literals are immutable; &str is an immutable reference.
    let mut string = s.to_string(); //&str to String
    string.push_str(" append");
    println!("{}", string);
}

//a string slice has static lifetime
let s = "hello";
//means
let s: &’static str = "hello";
```

## lifetimes are only about reference

a ref must die before its referent

in rust: 
- A resource can only have one owner at a time. When it goes out of the scope, Rust removes it from the Memory.

- When we want to reuse the same resource, we are referencing it/ borrowing its content.

- When dealing with references, we have to specify lifetime annotations to provide instructions for the compiler to set how long those referenced resources should be alive.

- ⭐ But because of lifetime annotations make the code more verbose, in order to make common patterns more ergonomic, Rust allows lifetimes to be elided/omitted in fn definitions. In this case, the compiler assigns lifetime annotations implicitly.


```rust
// No inputs, return a reference
fn function1<'a>() -> &'a str {}

// Single input
fn function2<'a>(x: &'a str) {}

// Single input and output, both have the same lifetime
// The output should live at least as long as input exists
fn function3<'a>(x: &'a str) -> &'a str {} // no need the lifetime annotation,lifetime elision

// Multiple inputs, only one input and the output share same lifetime
// The output should live at least as long as y exists
fn function4<'a>(x: i32, y: &'a str) -> &'a str {}

// Multiple inputs, both inputs and the output share same lifetime
// The output should live at least as long as x and y exist
fn function5<'a>(x: &'a str, y: &'a str) -> &'a str {}

// Multiple inputs, inputs can have different lifetimes 🔎
// The output should live at least as long as x exists
fn function6<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {}
```

## lifetimes in struct/enum
```rust
// Single element
// Data of x should live at least as long as Struct exists
struct Struct1<'a> {
    x: &'a str
}

// Multiple elements
// Data of x and y should live at least as long as Struct exists
struct Struct2<'a> {
    x: &'a str,
    y: &'a str
}

// Variant with a single element
// Data of the variant should live at least as long as Enum exists
enum Enum<'a> {
    Variant(&'a Type)
}
```
