---
title: jvm学习
comments: true
mp3: /music/blog.mp3
date: 2021-09-14 10:52:23
updated: 2021-09-14 10:52:23
tags:
categories:
  - JAVA
keywords: 
  - JVM
---


## JVM 结构
+ 类装载器子系统 Class Loader
+ 运行时数据区 Runtime Data Area
+ 执行引擎 Execution Engine
+ 本地方法接口 Native Interface
+ 本地方法库
## JVM内存结构
+ 虚拟机方法栈:每个方法的进入和返回，对应入栈和出栈
+ 本地方法栈:本地方法
+ 方法区:共享，类信息、常量、静态变量
+ 堆:共享，对象实例，最大的区域
+ 程序计数器:线程执行字节码指令

从JDK1.8开始，去掉方法区，而是直接用本地内存，分为元数据区和直接内存

## 垃圾回收算法
+ Mark-Sweep（标记-清除）算法
+ Copying（复制）算法
+ Mark-Compact（标记-整理）算法（压缩法）
+ Generational Collection（分代收集）算法
## 垃圾回收器
### Serial/Serial Old收集器
 是最基本最古老的收集器，它是一个单线程收集器，并且在它进行垃圾收集时，必须暂停所有用户线程。Serial收集器是针对新生代的收集器，采用的是Copying算法，Serial Old收集器是针对老年代的收集器，采用的是Mark-Compact算法。它的优点是实现简单高效，但是缺点是会给用户带来停顿。

### ParNew收集器
是Serial收集器的多线程版本，使用多个线程进行垃圾收集。

### Parallel Scavenge收集器
是一个新生代的多线程收集器（并行收集器），它在回收期间不需要暂停其他用户线程，其采用的是Copying算法，该收集器与前两个收集器有所不同，它主要是为了达到一个可控的吞吐量。

### Parallel Old收集器
是Parallel Scavenge收集器的老年代版本（并行收集器），使用多线程和Mark-Compact算法。

### CMS（Concurrent Mark Sweep）收集器
是一种以获取最短回收停顿时间为目标的收集器，它是一种并发收集器，采用的是Mark-Sweep算法。
### G1收集器
是当今收集器技术发展最前沿的成果，它是一款面向服务端应用的收集器，它能充分利用多CPU、多核环境。因此它是一款并行与并发收集器，并且它能建立可预测的停顿时间模型。
### ZGC
ZGC是从JDK11中引入的一种新的支持弹性伸缩和低延迟垃圾收集器，ZGC可以工作在KB~TB的内存之下，作为一种并发的垃圾收集器，ZGC保证应用延迟不会超过10毫秒(即便在堆内存很大的情况下)，并且可以将未提交的内存归还给操作系统。
ZGC最典型的特性是它是一款并发(concurrent)的GC，其它的特性如下：

+ 它可以标记内存，复制和迁移(relocate)内存，所有的操作都是并发的，同时它有一个并发的引用处理器
其它的垃圾收集器都是使用store barriers，ZGC使用load barriers，用于跟踪内存
    - lock->unlock->read->load  读内存
    - use->assign->store->write  写内存
+ ZGC可以更加灵活的配置大小和策略，相比于G1，它可以更好的处理非常大(very large)对象的释放
+ ZGC只有一代，没有新生代，老年代什么的，但是ZGC可以支持局部压缩，在内存恢复和迁移(reclaim and relocate)时，ZGC仍然有很高的性能
+ ZGC依赖NUMA-aware(非均衡存储器访问)，需要我们的内存支持这种特点