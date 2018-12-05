---
title: install go
date: 2018-11-06 19:56:00
tags: [go,install]
categories: go
---
# Linux 下安装
1. 下载
  ```bash
    wget https://dl.google.com/go/go1.11.linux-amd64.tar.gz
  ```
2. 解压到/usr/local目录下
```bash
 tar -C /usr/local -zxf go1.11.linux-amd64.tar.gz
```

3. 加入环境变量
```bash
export PATH=$PATH:/usr/local/go/bin
```

4. 验证
```bash
go version
```

# windows下安装

1、下载https://studygolang.com/dl
2、双击安装
3、默认情况下.msi文件会安装在 c:\Go 目录下。你可以将 c:\Go\bin 目录添加到 PATH 环境变量中。添加后你需要重启命令窗口才能生效。
