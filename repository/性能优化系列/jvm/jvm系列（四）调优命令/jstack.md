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

## 2.命令介绍

```shell
[root@bogon ~]# jstack -help
Usage:
    jstack [-l] <pid>
        (to connect to running process)
    jstack -F [-m] [-l] <pid>
        (to connect to a hung process)
    jstack [-m] [-l] <executable> <core>
        (to connect to a core file)
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>
        (to connect to a remote debug server)

Options:
    #当'jstack [-l] pid'没有相应信息的时候强制打印栈信息。
    -F  to force a thread dump. Use when jstack <pid> does not respond (process is hung)、
    #打印java和native c/c++框架的所有栈信息。
    -m  to print both java and native frames (mixed mode)
    #列表，打印关于锁的附加信息，例如属于java.util.concurrent的ownable synchronizers列表。
    -l  long listing. Prints additional information about locks
    #打印帮助信息
    -h or -help to print this help message

```

## 3.使用

### 3.1.死锁示例

```java
public class JStackDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new DeadLockClass(true));//建立一个线程
        Thread t2 = new Thread(new DeadLockClass(false));//建立另一个线程
        TimeUnit.SECONDS.sleep(1);
        t1.start();//启动一个线程
        t2.start();//启动另一个线程
    }
}

class DeadLockClass implements Runnable {

    private static final Object object1 = new Object();
    private static final Object object2 = new Object();
    private boolean flag;// 控制线程

    DeadLockClass(boolean flag) {
        this.flag = flag;
    }

    public void run() {
        if (flag) {
            while (true) {
                synchronized (object1) {
                    System.out.println("o1 " + Thread.currentThread().getName());
                    synchronized (object2) {
                        System.out.println("o2 " + Thread.currentThread().getName());
                    }
                }
            }
        } else {
            while (true) {
                synchronized (object2) {
                    System.out.println("o2 " + Thread.currentThread().getName());
                    synchronized (object1) {
                        System.out.println("o1 " + Thread.currentThread().getName());
                    }
                }
            }
        }
    }
}
```

我们使用jstack来看一下线程堆栈信息：

```shell
[root@bogon ~]# jstack 5552
2018-04-23 22:51:59
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.121-b13 mixed mode):

"DestroyJavaVM" #13 prio=5 os_prio=0 tid=0x0000000004952800 nid=0x478 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Thread-1" #12 prio=5 os_prio=0 tid=0x000000001ac2c800 nid=0x31e0 waiting for monitor entry [0x000000001b86f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.westboy.demo.DeadLockClass.run(JStackDemo.java:40)
        - waiting to lock <0x00000000d5ab1928> (a java.lang.Object)
        - locked <0x00000000d5ab1938> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

"Thread-0" #11 prio=5 os_prio=0 tid=0x000000001ac63000 nid=0x2d10 waiting for monitor entry [0x000000001b76e000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.westboy.demo.DeadLockClass.run(JStackDemo.java:31)
        - waiting to lock <0x00000000d5ab1938> (a java.lang.Object)
        - locked <0x00000000d5ab1928> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

"Service Thread" #10 daemon prio=9 os_prio=0 tid=0x000000001abb3800 nid=0x1ddc runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C1 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x000000001ab24800 nid=0x2994 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x000000001aaf7000 nid=0x1d28 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x000000001aaf4000 nid=0x19d0 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x000000001aaba800 nid=0x2568 runnable [0x000000001b16e000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:171)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
        - locked <0x00000000d5b32ea0> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(InputStreamReader.java:184)
        at java.io.BufferedReader.fill(BufferedReader.java:161)
        at java.io.BufferedReader.readLine(BufferedReader.java:324)
        - locked <0x00000000d5b32ea0> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(BufferedReader.java:389)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:64)

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x0000000019669000 nid=0x22c8 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001961f000 nid=0xa8c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x00000000195f9000 nid=0x3160 in Object.wait() [0x000000001a96e000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d5908ec8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:143)
        - locked <0x00000000d5908ec8> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:164)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:209)

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x0000000004a42000 nid=0x1968 in Object.wait() [0x000000001a86f000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x00000000d5906b68> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x00000000d5906b68> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

"VM Thread" os_prio=2 tid=0x00000000195d7000 nid=0x2cc8 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000000004968000 nid=0x2efc runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x0000000004969800 nid=0x1d48 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x000000000496b000 nid=0x11bc runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x000000000496c800 nid=0x247c runnable

"VM Periodic Task Thread" os_prio=2 tid=0x000000001abb7000 nid=0x22b8 waiting on condition

JNI global references: 33


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x0000000004a487c8 (object 0x00000000d5ab1928, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x0000000004a4c918 (object 0x00000000d5ab1938, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at com.westboy.demo.DeadLockClass.run(JStackDemo.java:40)
        - waiting to lock <0x00000000d5ab1928> (a java.lang.Object)
        - locked <0x00000000d5ab1938> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)
"Thread-0":
        at com.westboy.demo.DeadLockClass.run(JStackDemo.java:31)
        - waiting to lock <0x00000000d5ab1938> (a java.lang.Object)
        - locked <0x00000000d5ab1928> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```

Thread-1在想要执行第40行的时候，当前锁住了资源<0x00000000d5ab1928>,但是他在等待资源<0x00000007d6aa2c98> Thread-0在想要执行第31行的时候，当前锁住了资源<0x00000007d6aa2c98>,但是他在等待资源<0x00000007d6aa2ca8> 由于这两个线程都持有资源，并且都需要对方的资源，所以造成了死锁。 原因我们找到了，就可以具体问题具体分析，解决这个死锁了。


## 4.参考资料

* [Oracle官网关于jstack命令的使用介绍](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstack.html)
* [java命令--jstack 工具](https://www.cnblogs.com/kongzhongqijing/articles/3630264.html)
* [Java命令学习系列（二）——Jstack](http://www.hollischuang.com/archives/110)
#
———— ☆☆☆ —— 返回 -> [westboy-jvm系列（四）调优命令](index.md) <- 目录 —— ☆☆☆ ————