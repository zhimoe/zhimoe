---
title: 'git 常用命令备忘录'
date: "2019-09-01T21:01:06+08:00"
toc: true
categories:
 - "编程"
tags: 
  - git
  - code
--- 

记录日常开发中偶尔会遇到的但是总是记不住的git命令.
以下技巧都来自于[oh shit git](https://ohshitgit.com/) 和 [stackoverflow](https://stackoverflow.com). 版权归作者所有.

<!--more-->

## delete all history commit and commit current content
```bash
git checkout --orphan tmp_branch && git add -A && git commit -am "first commit" && git branch -D master && git branch -m master && git push -f origin master
```

## store password in local
```bash
git config credential.helper store
```

## git reflog
```bash
git reflog
# you will see a list of every thing you've
# done in git, across all branches!
# each one has an index HEAD@{index}
# find the one before you broke everything
git reset HEAD@{index}
# magic time machine
```

## git commit --amend
```bash
# make your change
git add . # or add individual files
git commit --amend --no-edit
# now your last commit contains that change!
# WARNING: never amend public(remote) commits!!!

# I need to change the message on my last commit!
git commit --amend 
# follow prompts to change the commit message
```

## undo a commit
```bash
# Oh shit, I need to undo a commit from like 5 commits ago!
# find the commit you need to undo
git log
# use the arrow keys to scroll up and down in history
# once you've found your commit, save the hash
git revert [saved hash]
# git will create a new commit that undoes that commit
# follow prompts to edit the commit message
# or just save and commit
```

## undo a file's changes
```bash
# find a hash for a commit before the file was changed
git log
# use the arrow keys to scroll up and down in history
# once you've found your commit, save the hash
git checkout [saved hash] -- path/to/file
# the old version of the file will be in your index
git commit -m "Wow, you don't have to copy-paste to undo"

```


## git stash
```bash
# 如果临时想要将代码恢复到最近一次commit,帮助同事复现他的问题
# 使用git stash 暂存当前修改,这个不是stage,也不是commit
git stash

# 显示当前暂存历史
git stash list

# 找回暂存
git stash apply
# or spec which the stash 
git stash apply stash@{1}

```

## git rebase

## git pull request

## git cherry-pick

## commit change in submodule
```bash
# submodule is a independent repo,
# so you need commit/push change in submodule first and then 
# update(commit) the main project to refer a new submodule commit hash

# step 1
cd path/to/submodule
git add <stuff>
git commit -m "comment"
git push

# step 2
cd /main/project
git add path/to/submodule
git commit -m "updated my submodule"
git push

```

