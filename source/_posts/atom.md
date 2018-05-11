---
title: 使用Atom编辑Markdown
date: 2018-05-10 14:58:49
tags: atom
categories: tool
---
## atom
工欲善其事必先利其器，找到一款好用的编辑器Atom。通过下载插件，可以用来编辑Markdown文本
### 1、安装Atom
下载地址：[https://atom.io/](https://atom.io/)


`Ctrl + Shift + P` 可以调出快捷键
### 2、安装插件
`File` -->  `Setting` --> `Install` --> Search for package
输入插件名，即可安装

## markdown 好用的插件
### 1、增强预览插件（markdown-preview-plus）
atom自带的Markdown预览插件markdown-preview功能比较简单，markdown-preview-plus做了功能扩展和增加。
* 支持预览实时渲染（ `Ctrl + Shift + M` ）
* 支持Latex公式（ `Ctrl + Shift + X` ）

### 2、粘贴图片 markdown-image-paste
* 截图到剪切板
* 在Markdown新起一行输入文件名
* Ctrl+V 会自动把图片保存到Markdown文件的相同目录下

### 3、表格编辑插件markdown-table-editor
使用方法：
* 先输入一个 | 和一些内容，按`tab`键，即可移动到下一列
* 按`enter`键，移动到下一行
* 按`esc`键，结束编辑表格

Commands:

| Commands      | Description           | Keybinding |
| ------------- | --------------------- | ---------- |
| Next Cell     | Move to next cell     | tab        |
| Previous Cell | Move to previous cell | shfit+ tab |
| Next Row      | Move to next row      | enter      |
| Escape        | Escape from the table | esc        |


如果你添加一些快捷键到你的 keymap.cson
``` bash
'atom-text-editor:not(.mini):not(.autocomplete-active).markdown-table-editor-active':
  'ctrl-left'           : 'markdown-table-editor:move-left'
  'ctrl-right'          : 'markdown-table-editor:move-right'
  'ctrl-up'             : 'markdown-table-editor:move-up'
  'ctrl-down'           : 'markdown-table-editor:move-down'
  'shift-ctrl-left'     : 'markdown-table-editor:align-left'
  'shift-ctrl-right'    : 'markdown-table-editor:align-right'
  'shift-ctrl-up'       : 'markdown-table-editor:align-center'
  'shift-ctrl-down'     : 'markdown-table-editor:align-none'
  'alt-shift-ctrl-left' : 'markdown-table-editor:move-column-left'
  'alt-shift-ctrl-right': 'markdown-table-editor:move-column-right'
  'alt-shift-ctrl-up'   : 'markdown-table-editor:move-row-up'
  'alt-shift-ctrl-down' : 'markdown-table-editor:move-row-down'
  'ctrl-k ctrl-i'       : 'markdown-table-editor:insert-row'
  'ctrl-k alt-ctrl-i'   : 'markdown-table-editor:delete-row'
  'ctrl-k ctrl-j'       : 'markdown-table-editor:insert-column'
  'ctrl-k alt-ctrl-j'   : 'markdown-table-editor:delete-column'
```

### 4、代码增强language-markdown
打开一个 Markdown 文件，按`ctrl+shift+L`选择“Markdown”

### 5、Markdown Maker
