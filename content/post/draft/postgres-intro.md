+++
title = 'Postgres 数据库入门'
date = '2023-10-01T19:02:16+08:00'
categories = ['编程']
tags = ['code','notes']
toc = true
draft = true
+++

postgres 数据库是目前使用第二广泛的开源数据库，并且由于良好的架构设计，可扩展性和可定制性远远优于 MySQL。本文只是一个应用开发人员基本的学习笔记，之前有简单的 MySQL 使用基础。

<!--more-->

## 安装
### 使用 docker 安装 server
```bash
docker run --rm -P -p 127.0.0.1:5432:5432 -e POSTGRES_PASSWORD="angus" --name pg postgres
```
### 安装客户端
macOS 可以通过 brew 安装 psql cli 工具而不是安装整个 postgres。
```bash
brew install libpq
# brew不会添加libq到path（如果安装postgresql则会），需要手工添加
echo 'export PATH="/opt/homebrew/opt/libpq/bin:$PATH"' >> ~/.zshrc && source !$
# 或者只添加 psql一个cli，其他bin下面的工具忽略
ln -s /opt/homebrew/opt/libpq/bin/psql /usr/local/bin/psql
```
连接数据库：
```bash
psql postgresql://postgres:angus@localhost:5432/postgres
```
也可以安装 dbeaver GUI 客户端。

## 和 mysql 的不同
[AWS: MySQL 和 PostgreSQL 有何区别](https://aws.amazon.com/cn/compare/the-difference-between-mysql-vs-postgresql/)

## 数据类型
数值类型 smallint、smallint、smallint、decimal、numeric、real、double precision、smallserial、serial、bigserial

货币类型 money

字符类型

日期类型

bool 类型 true false unknown

枚举类型 `CREATE TYPE week AS ENUM ('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun');`

集合类型 point  line lseg box path circle

网络地址 cidr inet macaddr

bit 串类型 bit 类型的数据必须准确匹配长度 n，试图存储短些或者长一些的数据都是错误的。bit varying 类型数据是最长 n 的变长类型；更长的串会被拒绝。写一个没有长度的 bit 等效于 bit(1)，没有长度的 bit varying 意思是没有长度限制。

文本类型 tsvector 的值是一个无重复值的 lexemes 排序列表，即一些同一个词的不同变种的标准化。
tsquery 存储用于检索的词汇，并且使用布尔操作符 &(AND)，|(OR) 和!(NOT) 来组合它们，括号用来强调操作符的分组。

uuid 类型 xml 类型 json 类型

数组类型 
```sql
CREATE TABLE sal_emp (
    name            text,
    pay_by_quarter  integer[], /* or: pay_by_quarter integer ARRAY[4], */
    schedule        text[][]
);
```