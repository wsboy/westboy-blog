# jstack(Java Stack Trace)

## 1.基本介绍

> jstack是java虚拟机自带的一种堆栈跟踪工具。

**主要分为两个功能**
1. 针对活着的进程做本地的或远程的线程dump； 
2. 针对core文件做线程dump。

jstack用于生成java虚拟机当前时刻的**线程快照**。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

> jstack命令主要用来查看Java线程的调用堆栈的，可以用来分析线程问题（如死锁）。

**线程状态**

想要通过jstack命令来分析线程的情况的话，首先要知道线程都有哪些状态，下面这些状态是我们使用jstack命令查看线程堆栈信息时可能会看到的线程的几种状态：

* NEW：未启动的。不会出现在Dump中。
* RUNNABLE：在虚拟机内执行的。
* BLOCKED：受阻塞并等待监视器锁。
* WATING：无限期等待另一个线程执行特定操作。
* TIMED_WATING：有时限的等待另一个线程的特定操作。
* TERMINATED：已退出的。






## 参考资料

* [Oracle官网关于jstack命令的使用介绍](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html)
* [java命令--jstack 工具](https://www.cnblogs.com/kongzhongqijing/articles/3630264.html)
* [Java命令学习系列（二）——Jstack](http://www.hollischuang.com/archives/110)
#
———— ☆☆☆ —— 返回 -> [westboy-jvm系列（四）调优命令](index.md) <- 目录 —— ☆☆☆ ————