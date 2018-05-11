---
title: 安装并使用hexo
date: 2018-05-12 00:28:29
tags: [hexo,git,nodejs]
categories: hexo
---

## 安装Hexo
Hexo是一个快速、简洁、高效的博客框架。Hexo使用[Markdown](https://daringfireball.net/projects/markdown/)解析文章。可以快速生成靓丽的主题的静态网页。

Hexo参考文档：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)

### 安装前提
安装Hexo前，确保您的电脑里已经安装了
> node.js

> git

如果你的电脑已经安装了上述必备程序，那么接下来只需要使用npm即可完成hexo的安装。
```bash
$ npm install -g hexo-cli
```

### 安装git
git参考文档：[https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2)

由于我们写博客一般都在windows系统上面，这里只写一下windows环境安装过程。

Linux下安装参考上面的文档。

点击[https://git-scm.com/download/win](https://git-scm.com/download/win),会自动下载安装程序，点下一步下一步就可以了。

### 安装node.js
windows下64位下载地址：
[http://nodejs.cn/download/](http://nodejs.cn/download/)

下载后 一直点下一步下一步即可。

安装完成后，cmd中输入path。
我们可以看到环境变量中已经包含了C:\Program Files\nodejs\

### 安装hexo
```bash
$ npm install -g hexo-cli
```

## 建站
安装后，执行下面的命令，Hexo将会在指定的文件夹中新建所需的文件。
```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
如果没有输入folder，则是在当前目录中新建。

新建完成后，目录结构如下所示
```bash
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
## 使用hexo
### 新建文章
```bash
$ hexo new "My New Post"    #可以简写成hexo n
```

### 生成静态网页
```bash
$ hexo generate
$ hexo g          #可以简写成hexo g
```

### 启动服务
```bash
$ hexo server
$ hexo s          #可以简写成hexo #!/bin/sh
$ hexo s --debug
```
启动完以后，可以通过 127.0.0.1:4000 访问

### 部署到远程服务器上
```bash
$ hexo deploy
$ hexo d         #可以简写成hexo d
```
注：通过配置github pages的地址到_config.yml中，执行这条命令后，可以一键部署到github pages。

### 清理
```bash
$ hexo clean
```
