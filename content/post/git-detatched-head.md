---
title: "Git 的detatched Head模式和解决问题方法"
date: "2021-03-09T15:49:09+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - git
---

有时候commit完代码后`git push`会遇到下面的错误
```shell
To push the history leading to the current (detached HEAD)
```
错误提示说当前HEAD没有指向任何分支，但是你记得明明有指向一个分支的

<!--more-->

### 复现问题
1. 假设你当前在master分支，且有两次提交
```shell
zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* 180d098 (HEAD -> master) 2nd commit
* ec6a47e start

```
2. 切回到`ec6a47e`这次提交
```shell
zhimoe@home:~/code/gittest$ git checkout ec6a
Note: switching to 'ec6a'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -c with the switch command. Example:

  git switch -c <new-branch-name>

Or undo this operation with:

  git switch -

Turn off this advice by setting config variable advice.detachedHead to false

HEAD is now at ec6a47e start

```
git直接会提示你当前HEAD已经detached。这是因为当HEAD离开当前分支（master）的末端commit时，Git会默认你想要离开当前分支，但是Git不会自动创建一个新分支（因为没有提供分支名称）。
所以HEAD变成没有指向任何分支的窘境，即使你再次回到刚才那个分支的末端commit，还是处于detached状态。
3. 切回master分支的末端commit并提交新内容
```shell
zhimoe@home:~/code/gittest$ git checkout 180d
Previous HEAD position was ec6a47e start
HEAD is now at 180d098 2nd commit

zhimoe@home:~/code/gittest$ echo "3nd file " > 3.txt
zhimoe@home:~/code/gittest$ git add . && git commit -m "3nd commit"
[detached HEAD fca1add] 3nd commit
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt
 
zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* fca1add (HEAD) 3nd commit
* 180d098 (master) 2nd commit
* ec6a47e start

```
可以看到此时HEAD和master分支还是分离的。

### 解决方法
1. 预防的方法就是没有commit的时候即使切回一个具体分支`git checkout master`
2. 如果已经提交了的话，给当前游离的commit创建一个分支，切换到该分支
2.1 
```shell
zhimoe@home:~/code/gittest$ git branch oops fca1add 

zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* fca1add (HEAD, oops) 3nd commit
* 180d098 (master) 2nd commit
* ec6a47e start

zhimoe@home:~/code/gittest$ git checkout oops
Switched to branch 'oops'

zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* fca1add (HEAD -> oops) 3nd commit
* 180d098 (master) 2nd commit
* ec6a47e start

```
2.2 使用rebase将oops分支接在master分支的末尾commit之后
```shell
zhimoe@home:~/code/gittest$ git rebase master
Current branch oops is up to date.

zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* fca1add (HEAD -> oops) 3nd commit
* 180d098 (master) 2nd commit
* ec6a47e start
# 看上去没有变化 回到master分支将oops内容merge进来

zhimoe@home:~/code/gittest$ git checkout master
Switched to branch 'master'

zhimoe@home:~/code/gittest$ git merge oops
Updating 180d098..fca1add
Fast-forward
 3.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 3.txt
 
zhimoe@home:~/code/gittest$ git log --oneline --graph --decorate
* fca1add (HEAD -> master, oops) 3nd commit
* 180d098 2nd commit
* ec6a47e start

```
最后删除oops分支：`git branch -d oops`.

参考
[一个完美的GitFlow模型](http://matrixzk.github.io/blog/20141104/git-flow-model/)
