# GC 调优

Sun JDK监控和故障处理命令有jps jstat jmap jhat jstack jinfo

- jps，JVM Process Status Tool,显示指定系统内所有的HotSpot虚拟机进程。
- jstat，JVM statistics Monitoring是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
- jmap，JVM Memory Map命令用于生成heap dump文件
- jhat，JVM Heap Analysis Tool命令是与jmap搭配使用，用来分析jmap生成的dump，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看
- jstack，用于生成java虚拟机当前时刻的线程快照。
- jinfo，JVM Configuration info 这个命令作用是实时查看和调整虚拟机运行参数。

常用调优工具分为两类,jdk自带监控工具：jconsole和jvisualvm，第三方有：MAT(Memory Analyzer Tool)、GChisto。

- jconsole，Java Monitoring and Management Console是从java5开始，在JDK中自带的java监控和管理控制台，用于对JVM中内存，线程和类等的监控
- jvisualvm，jdk自带全能工具，可以分析内存快照、线程快照；监控内存变化、GC变化等。
- MAT，Memory Analyzer Tool，一个基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗
- GChisto，一款专业分析gc日志的工具



## 在Java语言里，可作为GC Roots对象的包括如下几种： 

a.虚拟机栈(栈桢中的本地变量表)中的引用的对象 

b.方法区中的类静态属性引用的对象 

c.方法区中的常量引用的对象 

d.本地方法栈中JNI（Java Native Interface）的引用的对象

# Synchronized 结合 Java Object 对象中的 wait,notify,notifyAll

前面我们在讲 synchronized 的时候，发现被阻塞的线程什 么时候被唤醒，取决于获得锁的线程什么时候执行完同步代码块并且释放锁。那怎么做到显示控制呢？我们就需要借助一个信号机制：在 Object 对象中，提供了wait/notify/notifyall，可以用于控制线程的状态。

wait/notify/notifyall 基本概念
wait：表示持有对象锁的线程 A 准备释放对象锁权限，释放 cpu 资源并进入等待状态。

notify：表示持有对象锁的线程 A 准备释放对象锁权限，通知 jvm 唤醒某个竞争该对象锁的线程 X。 线程 A synchronized 代码执行结束并且释放了锁之后，线程 X 直接获得对象锁权限，其他竞争线程继续等待(即使线程 X 同步完毕，释放对象锁，其他竞争线程仍然等待，直至有新的 notify ,notifyAll 被调用)。

notifyAll：notifyall 和 notify 的区别在于，notifyAll 会唤醒所有竞争同一个对象锁的所有线程，当已经获得锁的线程 A 释放锁之后，所有被唤醒的线程都有可能获得对象锁权限。

需要注意的是：三个方法都必须在 synchronized 同步关键字所限定的作用域中调用，否则会报错 java.lang.IllegalMonitorStateException 。意思是因为没有同步，所以线程对对象锁的状态是不确定的，不能调用这些方法。另外，通过同步机制来确保线程从 wait 方法返回时能够感知到 notify 线程对变量做出的修改。

常见面试题：wait/notify/notifyall为什么需要在synchronized里面？

1.wait 方法的语义有两个，一个是释放当前的对象锁、另一个是使得当前线程进入阻塞队列， 而这些操作都和监视器是相关的，所以 wait 必须要获得一个监视器锁。

2.对于 notify 来说也是一样，它是唤醒一个线程，既然要去唤醒，首先得知道它在哪里?所以就必须要找到这个对象获取到这个对象的锁，然后到这个对象的等待队列中去唤醒一个线程。

3.每个对象可能有多个线程调用wait方法，所以需要有一个等待队列存储这些阻塞线程。这个等待队列应该与这个对象绑定，在调用wait和notify方法时也会存在线程安全问题所以需要一个锁来保证线程安全。

wait/notify 的基本原理
————————————————
版权声明：本文为CSDN博主「岁月安然」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/baidu_38083619/article/details/82527461