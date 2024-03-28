+++
title = "Git 的 detatched Head 模式和解决问题方法"
date = "2022-03-09T15:49:09+08:00"
categories = [ "编程",]
tags = [ "code", "git",]
toc = "true"
+++


有时候 commit 完代码后`git push`会遇到下面的错误
```shell
To push the history leading to the current (detached HEAD)
```
错误提示说当前 HEAD 没有指向任何分支，但是你记得明明有指向一个分支的

<!--more-->

### 复现问题
1、假设你当前在 master 分支，且有两次提交
```shell
Prj on  master
❯ git log --oneline --graph --decorate
* 314c9df (HEAD -> master) 2nd commit
* ae15845 initial commit
```
2、切回到第一次提交
```shell
Prj on  master
❯ git checkout ae15845
Note: switching to 'ae15845'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at ae15845 initial commit
```
git 直接会提示你当前 HEAD 已经 detached。这是因为当 HEAD 离开当前分支（master）的末端 commit 时，Git 会默认你想要离开当前分支，但是 Git 不会自动创建一个新分支（因为没有提供分支名称）。
所以 HEAD 变成没有指向任何分支的窘境，即使你再次回到刚才那个分支的末端 commit，还是处于 detached 状态。

3、切回 master 分支的末端 commit 并提交新内容
```shell
Prj (ae15845) # 注意 zsh配置这里展示的是当前HEAD，下面也给了提示
❯ git checkout 314c9df
Previous HEAD position was ae15845 initial commit
HEAD is now at 314c9df 2nd commit

# 提交点新东西
Prj (314c9df)
❯  echo "3nd file " > 3.txt

Prj (314c9df) [?]
❯ git add . && git commit -m "3nd commit"
[detached HEAD 09fb4a5] 3nd commit
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt

Prj (09fb4a5)
❯ git log --oneline --graph --decorate
* 09fb4a5 (HEAD) 3nd commit
* 314c9df (master) 2nd commit
* ae15845 initial commit

```
可以看到此时 HEAD 和 master 分支还是分离的。

### 真实场景复现
上面复现的方式很刻意，毕竟极少情况你会 checkout 一个具体的 commit 而不手动创建一个分支。日常工作中最可能遇到这个 detached HEAD 的场景是你使用`Git submodule`的时候。
敢说每个新手在使用 submodule 都会碰到 detached HEAD 问题。
原因是你的 submodule 没有记录正确的分支，即你在使用`git submodule add`时没有指定`-b <branch>`参数。

```shell
git submodule add -b main https://github.com/zhimoe/hugo-theme-next.git themes/next
```
或者直接在项目根目录下的`.gitmodules`文件中加上一行
```text
branch = main
# or branch = master
```

### 解决方法

1、预防的方法就是没有 commit 的时候及时切回一个具体分支`git checkout master`

2、如果已经提交了的话，给当前游离的 commit 创建一个分支，切换到该分支

```shell
Prj (09fb4a5)
❯ git branch oops 09fb4a5

Prj (09fb4a5)
❯  git log --oneline --graph --decorate
* 09fb4a5 (HEAD, oops) 3nd commit
* 314c9df (master) 2nd commit
* ae15845 initial commit

Prj (09fb4a5)
❯ git checkout oops
Switched to branch 'oops'

```

接着使用 rebase 将 oops 分支接在 master 分支的末尾 commit 之后
```shell
Prj on  oops
❯ git rebase master
Current branch oops is up to date.

Prj on  oops
❯ git log --oneline --graph --decorate
* 09fb4a5 (HEAD -> oops) 3nd commit
* 314c9df (master) 2nd commit
* ae15845 initial commit

Prj on  oops
❯ git checkout master && git merge oops
Switched to branch 'master'
Updating 314c9df..09fb4a5
Fast-forward
 3.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt

Prj on  master
❯ git log --oneline --graph --decorate
* 09fb4a5 (HEAD -> master, oops) 3nd commit
* 314c9df 2nd commit
* ae15845 initial commit
```
最后删除 oops 分支：`git branch -d oops`.

参考[一个完美的 GitFlow 模型](http://matrixzk.github.io/blog/20141104/git-flow-model/)
