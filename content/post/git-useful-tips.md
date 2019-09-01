---
title: 'git useful tips'
date: "2019-09-01T21:01:06+08:00"
toc: true
categories:
 - "编程"
tags: 
  - git
  - code
--- 

## delete all history commit and keep current content as commit
```bash
git checkout --orphan tmp_branch && git add -A && git commit -am "first commit" && git branch -D master && git branch -m master && git push -f origin master
```

## store password in local
```bash
git config credential.helper store
```

## git stash

## git rebase

## git pull request

## git cherry-pick