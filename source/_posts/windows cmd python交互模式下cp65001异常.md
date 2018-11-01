
---
title: windows cmdpython交互模式下cp65001异常
date: 2018-11-01 16:11:39
tags: [python]
categories: python
---
python安装后进入命令行交互模式，输入任何代码都报 `unknown encoding: cp65001`异常

需要将编码(UTF-8)修改为 简体中文(GBK)

在CMD窗口执行　`chcp 936`
