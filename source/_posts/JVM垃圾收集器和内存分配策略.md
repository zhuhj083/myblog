---
title: JVM垃圾收集器和内存分配策略
date: 2018-07-04 00:13:34
tags: [jvm,gc,java]
categories: java
---
# 概述

![图1](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/java%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA.PNG "图1")

* 程序计数器、虚拟机栈、本地方法栈 三个区域随线程而生，随线程而灭
* 栈中的栈帧随着方法的进入和退出而有条不紊地执行着出栈和入栈操作，每一个栈帧中分配多少内存基本上是在类结构确定下来时就已知
* 方法区和堆中的内存使用是不确定的，垃圾收集器所关注的就是这部分内存。

栈帧（Stack Frame） 是用于虚拟机执行时方法调用和方法执行时的数据结构，它是虚拟栈数据区的组成元素。每一个方法从调用到方法返回都对应着一个栈帧入栈出栈的过程。
一个线程中方法调用可能很长，很多方法都处于执行状态。对于执行引擎来说，只有处于栈顶的栈帧才是有效的，称为当前栈帧（Current Stack Frame），与之相关联的方法称为当前方法（Current Method）。
概念模型上，典型的栈帧主要由 局部变量表（Local Stack Frame）、操作数栈（Operand Stack）、动态链接（Dynamic Linking）、返回地址（Return Address）组成。


# 对象存活判定算法

## 引用计数算法
给每个对象添加一个引用计数器，每当有一个地方引用它时，计数器值加1，当引用失效时，计数器减1。

但是主流的Java虚拟机里并没有选用这种方法来管理内存，因为它很难解决对象之间相互循环引用的问题。

<!--more-->

## 可达性分析算法
主流的实现中，是通过可达性分析（Reachability Analysis）来判断对象是否存活。

基本思路：
通过一系列的成为“GC Roots”的对象作为起始点，从这些节点开始向下搜索，搜索所走的路径成为引用链（Reference Chain）。当一个对象到GC Roots没有任务引用链相连时，证明此对象不可用，就会被判定为可回收对象。

Java语言中，可作为GC Roots的对象包括下面几种：
* 虚拟机栈（栈帧中的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中JNI（即一般说的Native方法）引用的对象

## 引用
JDK1.2之后，引用分为4种，引用强度以此减弱：
1. 强引用（Strong Reference）:只要强引用在，垃圾收集器永远不会回收掉被引用的对象
2. 软引用（Soft Reference）：系统将要发生内存溢出之前，将会把这些对象列进回收范围之中进行第二次回收
3. 弱引用（Weak Reference）:被弱引用关联的对象只能生存到下一次垃圾收集发生之前
4. 虚引用（Phantom Reference）：唯一目的是能在这个对象被收集器回收时收到一个系统通知。

## 对象死亡的步骤
可达性分析算法中不可达的对象，会至少经历两次标记的过程，才会死亡：
1. 可达性分析后发现没有与GC Roots相连接的引用链，那么会被**第一次标记**并进行一次筛选
2. 帅选：如果对象没有覆盖finalize()方法或finalize()已经被虚拟机调用过，虚拟机会将不会再执行finalize()方法
3. 如果这个对象判断为有必要执行finalize()，那么这个对象会被放置在一个F-Queue队列中，稍后会由一个Finalizer线程去执行finalize()。finalize()方法是对象逃逸死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模标记。
4. 第二次标记时它被移除出“即将回收”的集合，这时候还没逃脱，就真的被回收了。

## 回收方法区
方法区在HotSpot虚拟机中的永久代，永久代的垃圾收集主要回收两部分内容：
1. 废弃常量
2. 无用的类
  * 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例
  * 加载该类的ClassLoader已经被回收
  * 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

# 新生代、老年代、永久代

### 新生代(堆中)
主要是用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁触发`MinorGC`进行垃圾回收。
新生代又分为Eden区、Survivor（ServivorFrom、ServivorTo）三个区(默认比例是8:1:1),采用`复制算法`。

### 老年代（堆中）
是存放那些在程序中经历了好几次回收仍然还活着或者特别大的对象（这个大就要看你是否设置了-XX：PretenureSizeThreshold 参数了）
老年代采用的是`标记-清除`或者`标记-整理`算法，这两个算法主要看虚拟机采用的哪个收集器，两种算法的区别是：标记-清除可能会产生大量连续的内存碎片。
在老年代中的GC则为`Major GC`。Major GC和Full GC会造成stop-the-world。

### 永久代
JVM的方法区，也被称为永久代。
在这里都是放着一些被虚拟机加载的类信息，静态变量，常量等数据。这个区中的东西比老年代和新生代更不容易回收

# 垃圾收集算法

## 标记-清除算法（Mark-Sweep）
分为“标记”和“清除”两个阶段。

不足：
效率问题，标记和清除的效率都不高
会产生大量的内存碎片

## 复制算法（Copying）
将内存按容量分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次性清理掉。这样使得每次都是对整个半区进行内存回收，内存分配时也不用考虑内存碎片等复杂情况。

现在商业虚拟机都采用这种算法来回收`新生代`。
不过是把内存分为一块较大的 Eden空间和两块较小的Survivor空间。 每次使用Eden和其中一块Survivor空间，当回收时，将Eden和Survivor中还存活的对象一次性复制到另外一块Surv空间上，最后清理掉Eden和刚才用过的Survivor空间。

如果另外一块Survivor空间没有足够的空间存放上一次新生代收集下来的存活对象，这些对象将直接通过分配担保机制进入`老年代`。

HotSpot虚拟机默认Eden和Survivor的大小比例是8:1。

## 标记-整理算法
标记过程仍然和“标记-清除”算法一样，但是后续不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

## 分代收集算法
根据对象存活周期的不同将内存分为几块：
一般把Java堆分为新生代和老年代。
新生代中，每次垃圾收集时，都会发现有大量对象死去，只有少量存活，那就选用复制算法。
老年代中，因为对象存活率高，没有额外空间对它进行分配担保，就必须使用“标记-清理”或"标记-整理"算法来进行回收。


# HotSpot的算法实现

## 枚举根节点
GC进行的时候，需要进行可达性分析，枚举根节点，可达性分析需要Stop The World。
可以作为GC Roots的节点主要在全局行的引用与执行上下文中。
当执行系统停顿下来后，虚拟机并不需要一个不漏地检查完所有执行上下文和全局的引用位置。HotSpot的实现中，使用一组称为OopMap的数据结构来直接得知哪些地方存着对象的引用。

## 安全点
在OopMap的协助下，HotSpot可以快速且准确地完成GC Roots枚举。HotSpot没有为每条指令都生成OopMap，只是在特定的位置记录下这些信息，这些位置称为安全点（Safepoint）。

程序执行时并非在所有地方都能停顿下来开始GC，只有在到达安全点时才能暂停。

## 安全区域
安全区域是指在一段代码片段之中，引用关系不会发生变化，在这个区域中的任意地方开始GC都是安全的。

线程要离开Safe Region时，它要检查系统是否已经完成了根节点枚举（或者是整个GC过程）。
