---
title: git submodule foreach git pull的问题
date: 2018-07-03 00:18:15
tags: [bug,git]
categories: bug
---
 git 执行 git submodule foreach git pull的时候，总是报下面的问题
 ```bash
 $ git submodule foreach git pull
Entering 'themes/next'
You are not currently on a branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

Stopping at 'themes/next'; script returned non-zero status.
 ```
后来进到子模块的目录下，执行 `git status`发现
```bash
$ git status
HEAD detached from 395bcfc
nothing to commit, working tree clean
```
貌似生成了一个临时的branch，执行`git branch -v`
```bash
zhuhaijun@ZhuHaijun-PC MINGW64 /f/myblog/themes/next ((63a1dd2...))
$ git branch -v
* (HEAD detached from 395bcfc) 63a1dd2 modified
  master                       395bcfc [behind 2] Update ISSUE_TEMPLATE.md
```
发现现在那个63a1dd2分支上。

**解决办法**：
直接用`git checkout master`切回master br吧，
然后别忘了用`git reset --hard 63a1dd2` 命令切到最新的hash啊。
