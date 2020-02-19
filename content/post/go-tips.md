---
title: "Go Tips"
date: "2020-02-09T17:12:41+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - go

draft: true
---

一些go相关文摘和tip

<!--more-->
[go best practices in 2016](https://peter.bourgon.org/go-best-practices-2016/)
1. Put all lib code in `pkg/subdirectory` folder and all binaries in `cmd`
2. Always use fully-qualified import paths. Never use relative imports.
3. [go build project layout](https://github.com/thockin/go-build-template)
4. Use struct literal initialization to avoid invalid intermediate state. Inline struct declarations where possible.
5. Make the zero value useful, especially in config objects
6. Loggers are dependencies, just like references to other components, database handles, commandline flags, etc.
7. Prefer go install to go build.
[more blogs on hist site](https://peter.bourgon.org/blog/)

## go code naming tips
- Structs are plain nouns: API, Replica, Object
- Interfaces are active nouns: Reader, Writer, JobProcessor
- Functions and methods are verbs: Read, Process, Sync