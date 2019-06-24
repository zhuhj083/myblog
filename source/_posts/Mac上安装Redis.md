---
title: Mac上安装Redis
date: 2019-06-24 19:40:51
tags: [mac,redis]
categories: Mac
---



# 安装redis 

redis安装有两种方式：下载源码编译安装、使用homebrew安装。 

通过homebrew安装redis 

```bash 

brew install redis 

```

# 配置文件 

安装完成后redis默认的配置文件redis.conf位于 

/usr/local/etc 

同时，redis-sentinel.conf也在这里。 

使用cat命令查看redis.conf： 

```bash 

cat /usr/local/etc/redis.conf 

```

<!--more-->

包含以下内容(删除了大部分内容)： 

```bash 

# Note that in order to read the configuration file, Redis must be 

# started with the file path as first argument: 

# 

# ./redis-server /path/to/redis.conf 

# Accept connections on the specified port, default is 6379 (IANA #815344). 

# If port 0 is specified Redis will not listen on a TCP socket. 

port 6379 

# TCP listen() backlog. 

tcp-backlog 511 

#The working directory 

dir /usr/local/var/db/redis/ 

```

根据以上内容,如果启动时不指定配置文件,redis会使用程序中内置的默认配置.但是只有在开发和测试阶段才考虑使用内置的默认配置，正式环境最好还是提供配置文件，并且一般命名为redis.conf 

# 启动redis-server 

如果需要给redis服务端指定配置文件，启动命令应该是这样的: 

```bash 

redis-server /usr/local/etc/redis.conf 

```

终端输出 

```bash 

46286:C 28 Feb 2019 11:36:55.966 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo 

46286:C 28 Feb 2019 11:36:55.966 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=46286, just started 

46286:C 28 Feb 2019 11:36:55.966 # Configuration loaded 

46286:M 28 Feb 2019 11:36:55.967 * Increased maximum number of open files to 10032 (it was originally set to 4864). 

_._ 

_.-``__ ''-._ 

_.-`` `. `_. ''-._ Redis 5.0.3 (00000000/0) 64 bit 

.-`` .-```. ```\/ _.,_ ''-._ 

( ' , .-` | `, ) Running in standalone mode 

|`-._`-...-` __...-.``-._|'` _.-'| Port: 6379 

| `-._ `._ / _.-' | PID: 46286 

`-._ `-._ `-./ _.-' _.-' 

|`-._`-._ `-.__.-' _.-'_.-'| 

| `-._`-._ _.-'_.-' | http://redis.io 

`-._ `-._`-.__.-'_.-' _.-' 

|`-._`-._ `-.__.-' _.-'_.-'| 

| `-._`-._ _.-'_.-' | 

`-._ `-._`-.__.-'_.-' _.-' 

`-._ `-.__.-' _.-' 

`-._ _.-' 

`-.__.-' 

46286:M 28 Feb 2019 11:36:55.968 # Server initialized 

46286:M 28 Feb 2019 11:36:55.969 * DB loaded from disk: 0.000 seconds 

46286:M 28 Feb 2019 11:36:55.969 * Ready to accept connections 

```

可以看出redis服务器启动成功，并在监听6379端口的网络连接。 

注意: 使用命令$ `redis-server`也可以启动,此时并不会加载任何配置文件,使用的是程序中内置(built-in)的默认配置. 

# 检查redis服务是否启动 

重新打开一个终端窗口，输入命令 

```bash 

redis-cli ping 

```

该终端输出 

```bash 

PONG 

```

说明服务器运作正常。 

# 关闭redis 

## 方法1 

在执行启动命令的终端窗口使用ctrl+c,此时第一个窗口输出 

```bash 

^C46286:signal-handler (1551325146) Received SIGINT scheduling shutdown... 

46286:M 28 Feb 2019 11:39:06.432 # User requested shutdown... 

46286:M 28 Feb 2019 11:39:06.432 * Saving the final RDB snapshot before exiting. 

46286:M 28 Feb 2019 11:39:06.434 * DB saved on disk 

46286:M 28 Feb 2019 11:39:06.434 * Removing the pid file. 

46286:M 28 Feb 2019 11:39:06.434 # Redis is now ready to exit, bye bye... 

zhuhaijundeMacBook-Pro:redis apple$ 

```

## 方法2 

在另外一个终端窗口执行`redis-cli shutdown`,此时第一个窗口输出 

```bash 

46288:M 28 Feb 2019 11:40:16.348 # User requested shutdown... 

46288:M 28 Feb 2019 11:40:16.348 * Saving the final RDB snapshot before exiting. 

46288:M 28 Feb 2019 11:40:16.349 * DB saved on disk 

46288:M 28 Feb 2019 11:40:16.349 * Removing the pid file. 

46288:M 28 Feb 2019 11:40:16.349 # Redis is now ready to exit, bye bye... 

```

