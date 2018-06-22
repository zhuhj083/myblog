---
title: Markdown 语法
date: 2018-06-22 12:02:07
tags: [markdown]
categories: tool
---

# Headers标题
```
#  H1
##  H2
###  H3
####  H4
#####  H5
######  H6

另外，H1和H2还能用以下方式显示：
一级标题
===

二级标题
---
```
# Emphasis 文本强调
```
*斜体* or _强调_
**加粗** or __加粗__
***粗斜体*** or ___粗斜体__
```
# Lists 列表
```
Unordered 无序列表：
* 无序列表
* 子项
* 子项

+ 无序列表
+ 子项
+ 子项

- 无序列表
- 子项
- 子项

Ordered 有序列表：
1. 第一行
2. 第二行
3. 第三行

1. 第一行
- 第二行
- 第三行

组合：
* 产品介绍（子项无项目符号）
    此时子项，要以一个制表符或者4个空格缩进

* 产品特点
    1. 特点1
    - 特点2
    - 特点3
* 产品功能
    1. 功能1
    - 功能2
    - 功能3

可有时我们会出现这样的情况，首行内容是以日期或数字开头：2013. 公司年度目标。
为了避免也被转化成有序列表，我们可以在"."前加上反斜杠（转义符）：2013\. 公司年度目标。
```

# Links 连接（title为可选项）
```
Inline-style 内嵌方式：
[link text](https://www.google.com "title text")

Reference-style 引用方式：
[link text][id]
[id]: https://www.mozilla.org "title text"

Relative reference to a repository file 引用存储文件：
[link text](../path/file/readme.text "title text")

还能这样使用：
[link text][]
[link text]: http://www.reddit.com

Email 邮件：
<example@example.com>
```

# Images 图片
```
Inline-style 内嵌方式：
![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "title text")

Reference-style 引用方式：
![alt text][logo]
[logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "title text"
```
# 代码
使用反引号(esc键下面的按钮)将代码包裹起来

# 分割线
   如果我们想用分割线对内容进行分割，我们可以在单独一行里输入3个或以上的短横线、星号或者下划线实现。短横线和星号之间可以输入任意空格。以下每一行都产生一条水平分割线。

# 引用
如果我们在文章中引用了资料，那么我们可以通过一个右尖括号">"来表示这是一段引用内容。我们可以在开头加一个，也可以在每一行的前面都加一个。我们还可以在引用里面嵌套其他的引用

# 换行
在需要换行的地方输入至少两个空格，然后回车即可

# 反斜杠
Markdown 可以利用反斜杠来插入一些在语法中有其它意义的符号，例如：如果你想要用星号加在文字旁边的方式来做出强调效果（但不用 <em\> 标签），你可以在星号的前面加上反斜杠：\\\*literal asterisks\\\*
Markdown 支持以下这些符号前面加上反斜杠来帮助插入普通的符号：
```
\   反斜线
`   反引号
*   星号
_   底线
{}  花括号
[]  方括号
()  括弧
#   井字号
+   加号
-   减号
.   英文句点
!   惊叹号
```
