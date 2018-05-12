---
title: 通过添加git子模块的方式来下载hexo next主题
date: 2018-05-12 17:26:07
tags: [submodule,hexo]
categories: hexo
---
# Fork
当我们使用hexo的时候，一般都会使用各种主题，大多数的文档都是叫我们直接`clone`主题到themes目录下，然后再配置`-config.yml`。

但是，当我们将我们的博客上传到我们自己的github上的时候，主题无法与博客数据一起同步。

原因是我们使用了`clone`将主题下载到本地，所以它自己本身也是一个`git`仓库。
因此它上层的博客仓库就无法对其进行管理。

更好的方法是`fork`一份主题到自己的github上，再将自己的github里的主题当作子模块加载进来。

`fork`的目的在于，我们可以对主题进行各种个性化的定制以及修改，
并对这些定制进行版本控制。同时，我们还能随时与原主题的更新进行合并。

1、fork `git@github.com:iissnan/hexo-theme-next.git` 到自己的github

# 通过git submodule add 添加为子模块

```bash
$ cd your-hexo-site
$ git submodule add https://github.com/someone/hexo-theme-next themes/next
```

运行 `git status`，会看到
```bash
$ git status
On branch master
Your branch is up to date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)
        new file:   .gitmodules
        new file:   themes/next
```

首先你注意到有一个`.gitmodules`文件。
```bash
$ cat .gitmodules
[submodule "themes/next"]
        path = themes/next
        url = https://github.com/iissnan/hexo-theme-next
```


# 修改themes/next主题
修改了`themes/next`里面的内容后，`git status`查看
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

# 更新submodule
submodule里面做了修改操作后，要在submodule和主目录下分别提交

## submodule push 到远端仓库
将添加的子模块push到远端submodule的仓库，分别执行
```bash
git add .
git commit -m '提交信息'
git push
```
## 主目录内push
然后再回到父目录,提交Submodule在父项目中的变动
```bash
git add .
git commit -m ' update submodule'
git push
```

# clone Submodule
* 方法一：一种是采用递归的方式clone整个项目
```bash
git clone git@github.com:***/***.git --recursive
```

* 方法二：一种是clone父项目，再更新子项目
clone我的博客项目后,执行下面更新命令
```bash
cd  your-hexo-site
git submodule init
git submodule update
```

# 更新submodule
方法一：在父目录直接执行
```bash
git submodule foreach git pull
```

方法二：在submodule目录下执行
```bash
git pull
```
