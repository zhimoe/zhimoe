---
title: "使用redis的hash优化内存使用[翻译]"
date: "2019-03-31T00:11:50+08:00"
toc: true
categories:
 - "翻译"
tags:
 - code
 - redis
---

使用redis的hash优化内存使用

# 原文
[Understanding Redis hash-max-ziplist-entries](https://www.peterbe.com/plog/understanding-redis-hash-max-ziplist-entries)

<!--more-->

# 问题和方案
场景: 有3亿张图片放在对象存储(DELL ECS/AMAZON EC2)上面,现在需要保存图片的id->用户id的映射.最直接的思路是:
```bash
set "media:1155220" "user1"
set "media:1155221" "user2"
```
这样设计key之后3亿张图片需要21GB的内存,因为redis的string是线性增长的.
此时可以使用hash优化内存使用.hash是类似java hashmap的数据结构: key field1 value1 field2 value2 ...
hash的强大在于它可以只获取一个field的value,而无需返回整个key.
再仔细想想,hash的key可以类比于分库分表的bucket概念.

回到上面的问题,Mike Krieger,Instagram的创始人[提出](https://engineering.instagram.com/storing-hundreds-of-millions-of-simple-key-value-pairs-in-redis-1091ae80f74c)将图片的id除以1000分片(sharding)到1000个hash key上:

```bash
HSET "mda-bkt:1155" "1155220" "user1" "1155221" "user2"
# mda-bkt:1155 是1155220/1000之后得到的bucket.
HGET "mda-bkt:1155" "1155220"
# 这里key的前缀*mda-bkt:)只重复了1000次,而上面的string方式重复了3亿次.
```
因为redis针对`hash list zset`三种结构使用了`ziplist`高效存储方案.

新的问题又来了,redis对于`ziplist`结构的key数量有限制的,即`hash-max-ziplist-entries`的含义是: 可使用内部空间优化存储的最多hash key

使用`ziplist`的数据结构有三个`list hash zset`:
```bash
list-max-ziplist-entries 512
list-max-ziplist-value 64
#Limits for ziplist use with LISTs.

hash-max-ziplist-entries 512
hash-max-ziplist-value 64
#Limits for ziplist use with HASHes (previous versions of Redis used a different name and encoding for this)
#hash-max-zipmap-entries 512 (for Redis < 2.6).

zset-max-ziplist-entries 128
zset-max-ziplist-value 64
#Limits for ziplist use with ZSETs.
```
你可以使用`debug_object(key)`查看你的key是否使用了`ziplist`结构.
建议`hash-max-ziplist-entries`最大设置为1000,过大会影响redis性能.

# 参考资料
[redis moemory optimize](https://redis.io/topics/memory-optimization)
[9.1.1 The ziplist representation-EBOOK – REDIS IN ACTION](https://redislabs.com/ebook/part-2-core-concepts/01chapter-9-reducing-memory-use/9-1-short-structures/9-1-1-the-ziplist-representation/)