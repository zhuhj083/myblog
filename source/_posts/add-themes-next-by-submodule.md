---
title: 通过添加git子模块的方式来下载hexo next主题
date: 2018-05-12 17:26:07
tags: [submodule,hexo]
categories: hexo
---
# 通过git submodule add 将外部项目添加为子模块
```bash
$ cd your-hexo-site
$ git submodule add https://github.com/iissnan/hexo-theme-next themes/next
```
运行 git status，还会看到
```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   .gitmodules
        new file:   themes/next
```
首先你注意到有一个.gitmodules文件。这是一个配置文件，保存了项目 URL 和你拉取到的本地子目录
```bash
$ cat .gitmodules
[submodule "themes/next"]
        path = themes/next
        url = https://github.com/iissnan/hexo-theme-next
```

# push到远端仓库
将添加的子模块push到远端仓库，分别执行
```bash
git add .
git commit -m '提交信息'
git push -u origin master
```

# 修改themes/next主题
修改了themes/next里面的内容后，git status 查看
```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   .gitmodules
        new file:   themes/next
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)
        modified:   .gitmodules
        modified:   themes/next (modified content, untracked content)
```
注：可以将每次修改的文件保存下来，在传到其他终端，覆盖其他终端的 themes/next内的相应文件

# 在B电脑上clone我的博客项目后
```bash
cd  your-hexo-site
git submodule init
git submodule update
```
