---
title: Git提交规范
date: 2020-04-01 15:52:02
categories:
- 笔记
tags: 
- git
---

保持干净的 git log

<!-- more -->

## 常见的 git log 污染
```bash
Merge remote-tracking branch 'origin/dev' into dev
Merge branch 'dev' of http://x.x.x.x/xxx/merge_or_rebase into dev
```

本地 git 仓库与远程仓库产生分叉时，git push 失败，提示 git pull
```bash
To http://x.x.x.x/xxx/merge_or_rebase.git
 ! [rejected]        dev -> dev (non-fast-forward)
error: failed to push some refs to 'http://xxx@x.x.x.x/xxx/merge_or_rebase.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

执行 git pull
git log 查看日志
```bash
Merge: fde641e b5bd4c9
Author: xxx <xxx@xxx.com>
Date:   Tue Mar 31 17:47:51 2020 +0800

    Merge branch 'dev' of http://x.x.x.x/xxx/merge_or_rebase into dev

```

执行 git fetch && git merge origin/dev
git log 查看日志
```bash
Merge: fde641e b5bd4c9
Author: xxx <xxx@xxx.com>
Date:   Tue Mar 31 17:50:24 2020 +0800

    Merge remote-tracking branch 'origin/dev' into dev

```

执行 git fetch && git rebase
git log 查看日志
```bash
DELL@DESKTOP-RT7532B MINGW64 ~/Desktop/merge_or_rebase (dev)
$ git fetch

DELL@DESKTOP-RT7532B MINGW64 ~/Desktop/merge_or_rebase (dev)
$ git rebase
First, rewinding head to replay your work on top of it...
Applying: update local.txt

DELL@DESKTOP-RT7532B MINGW64 ~/Desktop/merge_or_rebase (dev)
$ git log
commit d18ec408e2448bbeb1c0f3ead8c6d4c0478f1410 (HEAD -> dev)
Author: xxx <xxx@xxx.com>
Date:   Tue Mar 31 17:47:42 2020 +0800

    update local.txt

commit b5bd4c96864ddff87a80418ab980b29abf1e8fef (origin/dev)
Author: xxx <xxx@xxx.com>
Date:   Tue Mar 31 17:46:40 2020 +0800

    Add remote2 file

```
