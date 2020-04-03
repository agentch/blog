---
title: Git共享仓库
date: 2020-04-02 09:15:04
categories:
- 笔记
tags: 
- git
---

服务器创建git共享仓库

<!-- more -->

remote server
ip:192.168.5.11
```bash
$ git init --bare repo.git
```

local server
```bash
$ git init repo
$ cd repo
$ git remote add origin root@192.168.5.11
$ touch README.md
$ git add README.md
$ git commit -m 'init git'
$ git push -u origin master
```

remote server
```bash
$ git branch
* master
```
