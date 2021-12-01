---
title: "SpringBoot应用和Rust应用的Dockerfile最佳实践"
date: "2020-02-03T11:30:55+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - docker
 - spring
 - rust
---
记录spring boot和rust项目的Dockerfile的最佳实践.
<!--more-->

## spring boot应用Dockerfile
spring.io提供了一个boot应用的[Dockerfile](https://spring.io/guides/topicals/spring-boot-docker)指导.
不过有个问题,这个Dockerfile使用的maven是项目源码里面copy过去的.在一般项目中显然不规范.

Dockerfile的最终版:
```Dockerfile
# syntax=docker/Dockerfile:experimental
FROM maven:3-jdk-8-alpine as build
WORKDIR /workspace/app

COPY pom.xml .
COPY src src

RUN --mount=type=cache,target=/root/.m2  mvn package -DskipTests


# app base image
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ARG BUILD=/workspace/app/target

WORKDIR /app
COPY --from=build ${BUILD}/*.jar .
RUN jar -xf  ./*.jar
RUN rm ./*.jar

ENTRYPOINT ["java","-cp","/app","org.springframework.boot.loader.JarLauncher"]

```
要点:

1. `# syntax=docker/Dockerfile:experimental`表示启用docker实验特性BuildKit的mount cache功能,这样可以利用maven lib的cache提高镜像构建速度. 可以搜索docker BuildKit了解.
如果没有这一行,那么下面的`--mount=type=cache,target=/root/.m2`就是非法的. 由于是实验特性,构建镜像的时候需要设置一个环境变量`DOCKER_BUILDKIT=1`才能运行:` DOCKER_BUILDKIT=1 docker build -t zhimoe/boot-app .`
2. spring.io的教程里面使用的build镜像是`openjdk:8-jdk-alpine`,这个镜像是没有maven的,因为教程中的Dockerfile从源码复制了`mvnw,.mvn/`到镜像去.所以这里替换为`maven:3-jdk-8-alpine`
3. 使用了docker的[multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/)功能,`openjdk:8-jdk-alpine`由于没有maven,所以会比`maven`镜像少20M.
4. spring.io的教程里面在ENTRYPOINT里面是直接设置main class启动应用的. 这种硬编码方式不通用也不利于维护(修改main class name后Dockerfile也要修改).只要将应用的jar包解压出来的org目录(即org.springframework.boot.loader.jar解压内容,不到1M)保留,即可通过`org.springframework.boot.loader.JarLauncher`启动应用.
5. 注意`java -cp /app`中的classpath:`/app` 一定是绝对路径,否则java找不到main class,报错:`Error: Could not find or load main class org.springframework.boot.loader.JarLauncher`

## rust应用Dockerfile

```Dockerfile
# pull the latest version of Rust
FROM rust:latest AS builder

# create a new empty shell project
RUN USER=root cargo new --bin prj
WORKDIR /prj

# copy over your manifests
COPY ./Cargo.lock ./Cargo.toml ./

# change the crate.io source
COPY ./config $CARGO_HOME/

# this build step will cache your dependencies
RUN cargo build --release
RUN rm -r src/*

# copy your source files to WORKDIR/src
COPY ./src ./src
COPY ./static ./static

# build for release, note! the Cargo.toml package name in deps is _, not -
RUN rm ./target/release/deps/rs_notes*
RUN cargo build --release

RUN mv ./target/release/rs-notes .

## 2 stage build
# our final base
FROM debian:stretch-slim AS app


# for connecting to postgres and TLS hosts
# RUN apt update -y && apt install -y libpq-dev openssl libssl1.0-dev ca-certificates

# copy the build artifact and static resources from the build stage
COPY --from=builder /prj/rs-notes ./
COPY --from=builder /prj/static ./static

# set the startup command to run your binary
CMD ["./rs-notes"]

```

要点:
1. 如果使用scratch或者alpine镜像,那么需要将编译目标设置为`MUSL`,网络上有教程,个人感觉不需要.rust应用使用debian-slim基本在60M左右,只有spring boot应用镜像的一半大小.
2. 在国内由于网络问题,所以修改了cargo的crate.io mirror地址:`COPY ./config $CARGO_HOME/`. config内容如下:
```ini
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

```
3. build中使用了cargo缓存,即先将项目Cargo.toml和Cargo.lock复制到一个空项目中编译,然后再将源码复制进去编译.
4. `RUN rm ./target/release/deps/rs_notes*`,注意这里的`rs_notes`是下划线.cargo中package name转换为crate name的默认规则.