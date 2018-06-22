---
title: volatile and synchronized
date: 2018-06-22 14:50:03
tags: [volatile synchronized concurrency]
categories: java
---
# volatile概述
volatile是轻量级的synchronized,在多处理器开发中保证了共享变量的可见性。
可见性：当一个线程修改一个共享变量时，另一个线程能够读到这个修改的值。
volatile不会引起线程的上下文切换和调度。

如果一个字段被声明为volatile,Java线程内存模型确保所有线程看到这个变量的值是一致的。

## CPU术语定义
* 内存屏障（memory barriers）：是一组处理器命令，用于实现对内存操作的顺序限制
* 缓冲行（cache line）：CPU高速缓存中可以分配的最小存储单位 。处理器填写缓存行时会加载整个缓存行，现代CPU需要执行几百次CPU指令
* 原子操作(atomic operations):不可中断的一个活或一些列操作
* 缓存行填充(cache line fill):当处理器识别到从内存中读取操作数是可缓存的，处理器读取整个高速缓存行到适当的缓存（L1、L2、L3的或所有）
* 缓存命中（cache hit）:如果进行高速缓存行填充的内存位置仍然是下次处理器访问的地址时，处理器从缓存中读取操作数，而不是从内存中读取
* 写命中（write hit）：当处理器将操作数写回到一个内存缓存的区域时，它首先会检查这个缓存的内存地址是否在缓存中，如果存在一个有效的缓存行，则处理器将这个操作数写回到缓存，而不是写回到内存，这个操作被称为写命中
* 写缺失（write misses the cache）：一个有效的缓存行被写入到不存在的内存区域

# volatile实现原理
有volatile修饰的共享变量进行写操作的时候，会多出有Lock前缀的指令：
  1. Lock前缀指令会引起处理器缓存写回到内容
    * 执行指令期间，声言处理器的LOCK#信号，LOCK#信号确保在声言该信号期间，处理器可以独占任何共享内存
  2. 一个处理器的缓存写回到内存会导致其他处理器的缓存无效
    *  处理器使用嗅探技术保证它的内部缓存、系统内存和其他处理器的缓存的数据在总线上保持一致

# synchronized
synchronized实现同步的基础：Java中每一个对象都可以作为锁，具体表现为以下3种形式：
  * 对于普通同步方法，锁是当前实例对象
  * 对于静态同步方法，锁是当前类的Class对象
  * 对于同步方法块，锁是synchronized括号里配置的对象

synchronized在JVM里的实现原理是， JVM基于进入和退出Monitor对象来实现方法同步和代码块同步。
monitorenter指令在编译后插入到同步代码块的开始位置，monitorexit是插入到方法结束和异常处。
线程执行到monitorexter指令时，将会尝试获取对象所对应的monitor的所有权，即尝试获得对象的锁。

## Java对象头
synchronized用的锁是存在Java对象头里的`Mark Word`。
Java对象头的长度

| 长度     | 内容                   | 说明                             |
| -------- | ---------------------- | -------------------------------- |
| 32/64bit | Mark Word              | 存储对象的hashCode或锁信息等     |
| 32/64bit | Class Metadata Addredd | 存储到对象类型数据的指针         |
| 32/64bit | Array length           | 数组的长度（如果当前对象是数组） |

## 锁的升级与对比
锁一共4种状态，级别从低到高依次是：
* 无锁状态
* 偏向锁状态
* 轻量级锁状态
* 重量级锁状态

## 偏向锁
对于经常由同一线程多次获得锁的情况，为了让线程获得锁的代价更低，引入偏向锁。
偏向锁会在对象头和栈帧中的锁记录存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁
偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁