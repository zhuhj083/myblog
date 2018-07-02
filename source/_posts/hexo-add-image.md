---
title: hexo发布博文，添加图片
date: 2018-07-03 00:31:38
tags: hexo
categories: hexo
---

# 方法一

这个方法并不是很满意，采用的是方法二

**以下是具体步骤：**

1.修改hexo博客项目根目录_config.yml配置文件post_asset_folder项为true。

2、安装插件，在博客项目根目录下执行`npm install https://github.com/CodeFalling/hexo-asset-image --save`

2.hexo new “new blog”,新建一篇博客。

3.在source/_post文件夹里面就会出现一个“new-blog.md”的文件和一个“new-blog”的文件夹。

4、将图片复制到这个文件夹下，例如test.png

5.引用图片：
{% asset_img test.png [图片描述] %}

# 方法二
在github上建立一个仓库，专门用来存放图片。
将需要的图片上传到这个仓库

用以下的方式引入图片
```
![Alt text](/path/to/img.jpg "Optional title")
```
