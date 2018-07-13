---
title: mongodb replica sets副本集集群搭建
date: 2018-07-13 11:55:10
tags: [mongodb,replica sets,nosql]
categories: [mongodb]
---

# 一、replica sets简介
一个副本集是一组包含相同数据集的mongodb实例组成的集群系统。有一个primary节点，其他的节点为secondary节点。和主从复制的原理一样，副本集也是通过读取oplog来进行数据传输，oplog是一固定大小的表，创建的时候需要指定其大小，当oplog满的时候会删除旧的数据，所以设置其大小非常重要，如果oplog被primary节点覆盖而尚未被sencondary节点读取同步的话就要重新resync。
一般的replica sets集群的架构如下图所示：一主一备和一个仲裁（Arbitry）节点，仲裁节点不存放数据。

![replica sets](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/mongo.PNG "replica sets")

同步方式采用异步同步的方式，成员节点每隔2s发送一次heartbeat（pings）。当主节点与其他成员节点通信超时10s后，一个sencondary节点将会被选举为primary节点。

<!--more-->

# 二、环境搭建
1. 在所有机器上安装mongodb，我安装在/usr/local/ 目录下。
2. 创建数据目录，/search/odin/data
3. 在每台机器的mongodb的bin目录下创建配置文件mongod.conf,内如如下

  ```bash
  dbpath=/search/odin/mongodb/data
  logpath=/search/odin/mongodb/log
  fork=true
  port=27017
  oplogSize=2048
  ```

4. 启动mongodb

  ```bash
    ./mongod -f mongod.conf
  ```

# 三、创建replica sets
下面是搭建一个一主一从一仲裁 三节点replica sets的具体步骤

## 添加replSet参数
```bash
dbpath=/search/odin/mongodb/data

logpath=/search/odin/mongodb/log

fork=true

port=27017

replSet=rs0
```

然后启动三台服务器。

## 初始化副本集
连接其中一个节点，初始化命令只能执行一次。
可以先配置一个配置文件,然后使用rs.initiate(rsconf)来初始化,例如:
```bash

rsconf = {
        _id: "rs0",
           members: [
                      {
                       _id: 0,
                       host: "10.143.40.142:27017"
                      }
                  ]
}

rs.initiate(rsconf)

```

## 检查初始化配置文件
```bash
rs.conf()
```

## 添加secondary和arbitry节点
```bash

# 添加一个节点
rs.add("10.143.41.140:27017")

#添加一个arbitary节点
rs.add("10.143.55.191:27017"，true)

#查看副本集的当前状态：
rs.status()

```

## rs.remove删除一个结点
```bash
#rs.remove()就一个参数hostname:
rs.remove("mongodb3:27017")
```
## rs.addArb添加投票节点
rs.addArb()同样可以添加投票节点,也只有一个参数为hostname:
```bash
rs.remove("mongodb3:27017")
```


## 查看状态
```
db.printSlaveReplicationInfo();
```

## 查看数据信息

rs.isMaster()，是否为主节点

rs.secondary():是否为从节点

rs.primary():指出当前副本集中的主节点位于哪个进程

 rs.config():查看详细的配置信息
