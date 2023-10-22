+++
title = "SpringBoot 应用和 Rust 应用的 Dockerfile 最佳实践"
date = "2020-02-03T11:30:55+08:00"
categories = [ "编程",]
tags = [ "code", "docker", "spring", "rust",]
toc = "true"
+++


记录 spring boot 和 rust 项目的 Dockerfile 的最佳实践。

## spring boot 应用 Dockerfile
spring.io 提供了一个 boot 应用的[Dockerfile](https://spring.io/guides/topicals/spring-boot-docker)指导。
不过有个问题，这个 Dockerfile 使用的 maven 是项目源码里面 copy 过去的。在一般企业项目中这么做显然不规范，直接使用 maven 基础镜像更合理。

<!--more-->

Dockerfile 的最终版：
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
要点：

1. `# syntax=docker/Dockerfile:experimental`表示启用 docker 实验特性 BuildKit 的 mount cache 功能，这样可以利用 maven lib 的 cache 提高镜像构建速度。可以搜索 docker BuildKit 了解。
如果没有这一行，那么下面的`--mount=type=cache,target=/root/.m2`就是非法的。由于是实验特性，构建镜像的时候需要设置一个环境变量`DOCKER_BUILDKIT=1`才能运行：` DOCKER_BUILDKIT=1 docker build -t zhimoe/boot-app .`
2. spring.io 的教程里面使用的 build 镜像是`openjdk:8-jdk-alpine`，这个镜像是没有 maven 的，因为教程中的 Dockerfile 从源码复制了`mvnw和.mvn/`到镜像去。所以这里替换为`maven:3-jdk-8-alpine`
3. 使用了 docker 的[multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/)功能，`openjdk:8-jdk-alpine`由于没有 maven，所以会比`maven`镜像少 20M.
4. spring.io 的教程里面在 ENTRYPOINT 里面是直接设置 main class 启动应用的。这种硬编码方式不通用也不利于维护 (修改 main class name 后 Dockerfile 也要修改).只要将应用的 jar 包解压出来的 org 目录 (即 org.springframework.boot.loader.jar 解压内容，不到 1M) 保留，即可通过`org.springframework.boot.loader.JarLauncher`启动应用。
5. 注意`java -cp /app`中的 classpath:`/app` 一定是绝对路径，否则 java 找不到 main class，报错：`Error: Could not find or load main class org.springframework.boot.loader.JarLauncher`

## rust 应用 Dockerfile

```Dockerfile
# pull the latest version of Rust
FROM rust:latest AS builder

# create a new empty shell project
RUN USER=root cargo new --bin prj
WORKDIR /prj

# copy the manifests to WORKDIR/src
COPY ./Cargo.lock ./Cargo.toml ./
# change the crate.io source url if in China mainland
COPY ./config $CARGO_HOME/

# build without project source code
# this build step will cache your dependencies
RUN cargo build --release

# remove the WORKDIR/src
RUN rm -r src/*
# copy your source files to WORKDIR/src
COPY ./src ./src
COPY ./static ./static

# build for release, 
# note! the Cargo.toml package name in deps is _, not -
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

要点：
1. 如果使用 scratch 或者 alpine 镜像，那么需要将编译目标设置为`MUSL`，网络上有教程，感觉不需要考虑这个体积问题。rust 应用使用 debian-slim 基本在 60M 左右，只有 spring boot 应用镜像的一半大小。
2. 在国内由于网络问题，所以修改了 cargo 的 crate.io mirror 地址：`COPY ./config $CARGO_HOME/`. config 内容如下：
```ini
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

```
3. build 中使用了 cargo 缓存，即先将项目 Cargo.toml 和 Cargo.lock 复制到一个空项目中编译，然后再将源码复制进去编译。
4. `RUN rm ./target/release/deps/rs_notes*`，注意这里的`rs_notes`是下划线，是 cargo 中 package name 转换为 crate name 的默认规则。