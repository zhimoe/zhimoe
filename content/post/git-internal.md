+++
title = '[翻译]Git内部原理图解——对象、分支以及如何从零开始建仓库'
date = '2023-08-06T16:06:23+08:00'
categories = ['编程']
tags = ['git','翻译']
toc = true
+++

我们中的许多人每天都在使用 git，但是有多少人知道它的内部是怎么运作的呢？

例如我们使用 git commit 时发生了什么？提交（commit）与提交之间保存的是什么？两次提交之间难道只是文件的差异（diff）吗？如果是，这个差异是如何编码的？还是说每次提交都会保存一个当前仓库的完整快照（snapshot）呢？我们使用 git init 时到底发生了什么？

发现一篇非常精彩的Git内部原理文章[Git内部原理图解——对象、分支以及如何从零开始建仓库](https://medium.com/swimm/a-visualized-intro-to-git-internals-objects-and-branches-68df85864037)，[中文翻译](https://www.freecodecamp.org/chinese/news/git-internals-objects-branches-create-repo/)。文章作者甚至制作了[配套讲解视频](https://www.youtube.com/playlist?list=PL9lx0DXCC4BNUby5H58y6s2TQVLadV8v7)


<!--more-->

### Git对象
git内部有三种对象：
1. blob: 文件的内容，不包含metadata信息（创建时间，修改时间，作者等）
2. tree: 一个目录，包含blobs或者trees
3. commit: a snapshot of the working tree，一个tree的快照。 

三种git对象都是通过SHA-1哈希值来唯一标识，如下图所示。每个commit对象中，对于tree里面那些没有改动的内容，继续通过原hash引用。

![git 对象以及关系示意图](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/git-objects.1pz0i807ve0w.webp)

### 分支
A branch is just a named reference to a commit.

在上面的图片中，可以通过哈希值来引用一个commit，但是不方便，所以分支用来引用commit。可以理解为分支是一个指针，指向一个commit，一般默认是指向最后一个commit（也可以不是最后一个commit）。

git通过`HEAD`指针来确认当前所在分支。`HEAD`指针其实是`.git`目录下的一个`HEAD文件`,内容如下
```shell
> cat .git/HEAD
ref: refs/heads/master
```

### git如何记录变化

1. repository是一系列commit的集合
2. working dir是一个包含`.git`的目录
3. staging area是存放那些被git跟踪但是没有commit的内容

三者的关系如下图所示
![git-repo-workingdir](https://jsd.cdn.zzko.cn/gh/zhimoe/zhimoe.pic@main/pic/git-repo-workingdir.39ykllsr2to0.webp)

### git底层命令(plumbing)和上层命令(porcelain)
区分 底层（plumbing） 和 上层（porcelain） 两类 git 命令会对你很有帮助。这两个术语的应用奇怪地来自于马桶（没错，就是🚽）。马桶通常是用陶瓷（porcelain）做的，它的基本结构是管道（plumbing，上水道和下水道）。
上层命令就是`git init、git add、 git commit`等,下面介绍一下底层命令.

```shell
# 创建git对象
>echo "git is awesome" | git hash-object --stdin -w
# 查看.git目录的变化
>tree .git
# 查看一个git object类型 -t type
>git cat-file -t [obj-hash]
# blob|tree|commit
# 查看一个git object内容 -p pretty-print
>git cat-file -p [obj-hash]
# 添加object到staging area
>git update-index --add --cacheinfo 100644 <blob-hash> <filename>

# 创建一个tree对象,在tree对象中记录index内容
>git write-tree

# 为tree对象创建一个commit对象
>git commit-tree <tree-hash> -m <commit message>

```
git 实际上是使用 SHA-1 哈希值的前两个字符作为目录的名字，剩余字符用作 blob 所在文件的文件名。

### .git目录
一个`.git`目录至少包含三个内容
```shell
✔ tree .git
.git
├── HEAD        当前指向分支，默认内容是 ref: refs/heads/main
├── objects     git对象 blob、tree、commit的一种，其中对象的hash值前两个字符用于目录名，剩余的用于对象名
└── refs        分支和tag
    └── heads   当前working dir所有分支 默认分支不展示，只有多于一个分支才会展示 
```
添加一个文件并commit，然后创建一个新分支，再次检查.git目录
```shell
✔ tree .git
.git
├── HEAD
├── index
├── objects
│        ├── 8d
│        │   └── 0e41234f24b6da002d962a26c2495ea16a425f
│        ├── af
│        │   └── 7e0d93b83f49f601f5ef35edf5f9330fb4d7fd
│        └── c8
│            └── bcfef1da123a980537a5fa4cf9b7c4f387d451
└── refs
    └── heads
        ├── main
        └── test_branch

7 directories, 7 files
```
上面删除了logs目录,index文件保存的是staging area信息。打印objects目录下的三个文件
```shell
✔ git cat-file -p 8d0e41234f24b6da002d962a26c2495ea16a425f
hello git

 :~/code/temp (main)
✔ git cat-file -p c8bcfef1da123a980537a5fa4cf9b7c4f387d451
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f	file.txt

 :~/code/temp (main)
✔ git cat-file -p af7e0d93b83f49f601f5ef35edf5f9330fb4d7fd
tree c8bcfef1da123a980537a5fa4cf9b7c4f387d451
author zhimoe <xx@gmail.com> 1691313901 +0800
committer zhimoe <xx@gmail.com> 1691313901 +0800

first commit
```
可以看到分别是一个blob对象（file.txt)、一个tree对象和一个commit对象，后者依次引用前者。

### 参考
文章里面提到了很多git内部原理和概念:
[Git 内部原理 - 底层命令与上层命令](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BA%95%E5%B1%82%E5%91%BD%E4%BB%A4%E4%B8%8E%E4%B8%8A%E5%B1%82%E5%91%BD%E4%BB%A4)
[Git 内部原理 - Git 对象](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)