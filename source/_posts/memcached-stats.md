---
title: memcached启动与查看状态
date: 2018-05-10 14:58:49
tags: memcached
categories: memcached
---
##启动memcached
``` bash
memcached -d -p 11311 -u odin -m 512 -c 1024
```
-p <num>  设置TCP端口号，默认为11211
-U <num>  设置UDP端口号
-u <username> 指定用于运行进程的用户
-m  允许的最大内存量，单位是M，默认是64M
-c  最大连接数,默认为1024

## 连接memcached 和退出
``` bash
telnet 127.0.0.1 11211
quit
```

## stats查看状态
示例如下
``` bash
STAT pid 22459                             进程ID
STAT uptime 1027046                        服务器运行秒数
STAT time 1273043062                       服务器当前unix时间戳
STAT version 1.4.4                         服务器版本
STAT libevent 2.0.21-stable
STAT pointer_size 64                       操作系统字大小(这台服务器是64位的)
STAT rusage_user 0.040000                  进程累计用户时间
STAT rusage_system 0.260000                进程累计系统时间
STAT curr_connections 10                   当前打开连接数
STAT total_connections 82                  曾打开的连接总数
STAT connection_structures 13              服务器分配的连接结构数
STAT reserved_fds 20
STAT cmd_get 54                            执行get命令总数
STAT cmd_set 34                            执行set命令总数
STAT cmd_flush 3                           指向flush_all命令总数
STAT get_hits 9                            get命中次数
STAT get_misses 45                         get未命中次数
STAT delete_misses 5                       delete未命中次数
STAT delete_hits 1                         delete命中次数
STAT incr_misses 0                         incr未命中次数
STAT incr_hits 0                           incr命中次数
STAT decr_misses 0                         decr未命中次数
STAT decr_hits 0                           decr命中次数
STAT cas_misses 0                          cas未命中次数
STAT cas_hits 0                            cas命中次数
STAT cas_badval 0                          使用擦拭次数
STAT touch_hits 0
STAT touch_misses 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 15785                      读取字节总数
STAT bytes_written 15222                   写入字节总数
STAT limit_maxbytes 67108864               分配的内存数（字节）
STAT accepting_conns 1                     目前接受的链接数
STAT listen_disabled_num 0
STAT time_in_listen_disabled_us 0
STAT threads 4                             线程数
STAT conn_yields 0
STAT hash_power_level 16
STAT hash_bytes 524288
STAT hash_is_expanding 0
STAT malloc_fails 0
STAT conn_yields 0
STAT bytes 0                               存储item字节数
STAT curr_items 0                          item个数
STAT total_items 34                        item总数
STAT expired_unfetched 0
STAT evicted_unfetched 0
STAT evictions 0                           为获取空间删除item的总数
STAT reclaimed 0
STAT crawler_reclaimed 0
STAT crawler_items_checked 0
STAT lrutail_reflocked 0
```

## 获取运行状态
``` bash
echo stats | nc 127.0.0.1 11211
```
也可以查看到 stats

## 查看memcached的连接数
``` bash
netstat -nat | fgrep 11211 |awk '{print $4}' | fgrep 11211 | wc -l
```
