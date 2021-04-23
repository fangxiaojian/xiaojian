## synchronized的使用

在多线程并发编程中synchronized一直是元老级角色，很多人都会称呼它为重量级锁。但是，随着Java SE 1.6对 synchronized进行了各种优化之后，有些情况下它就并不那么重了，Java SE 1.6中为了减少获得锁和释放锁带来的 性能消耗而引入的偏向锁和轻量级锁，以及锁的存储结构和升级过程。通过 synchronized关键字来修饰在inc的方法上,看看执行结果。（可以自己尝试将synchronizrd去掉，看看结果得到的是不是1000）

```java

public class Demo{
    private static int count=0;
    public static void inc(){
        synchronized (Demo.class) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            count++; 
        }
    }
    public static void main(String[] args) throws InterruptedException {
        for(int i=0;i<1000;i++){
            new Thread(()->Demo.inc()).start();
        }
        Thread.sleep(3000);
        System.out.println("运行结果"+count); 
    }
}

```

## synchronized的三种应用方式

synchronized有三种方式来加锁，分别是

1. 修饰实例方法，作用于当前实例加锁，进入同步代码前要获得当前实例的锁
2. 静态方法，作用于当前类对象加锁，进入同步代码前要获得当前类对象的锁
3. 修饰代码块，指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

## synchronized括号后面的对象

synchronized 扩后后面的对象是一把锁，在java中任意一个对象都可以成为锁，简单来说，我们把 object 比喻是一 个 key，拥有这个 key 的线程才能执行这个方法，拿到这个 key 以后在执行方法过程中，这个 key 是随身携带的，并且只有一把。如果后续的线程想访问当前方法，因为没有key所以不能访问只能在门口等着，等之前的线程把key放回 去。所以，synchronized 锁定的对象必须是同一个，如果是不同对象，就意味着是不同的房间的钥匙，对于访问者 来说是没有任何影响的

## synchronized的字节码指令

通过 `javap -v` 来查看对应代码的字节码指令，对于同步块的实现使用了`monitorenter` 和 `monitorexit` 指令，前面我们在讲JMM的时候，提到过这两个指令，他们隐式的执行了 Lock 和 UnLock 操作，用于提供原子性保证。 `monitorenter` 指令插入到同步代码块开始的位置、`monitorexit` 指令插入到同步代码块结束位置，jvm 需要保证每个 `monitorenter` 都有一个 `monitorexit` 对应。

这两个指令，本质上都是对一个对象的监视器 (monitor) 进行获取，这个过程是排他的，也就是说同一时刻只能有 一个线程获取到由 synchronized 所保护对象的监视器线程执行到 monitorenter 指令时，会尝试获取对象所对应的 monitor 所有权，也就是尝试获取对象的锁;而执行 monitorexit，就是释放 monitor 的所有权。

对象的监视器（monitor）锁对象由 ObjectMonitor 对象实现（C++），其跟同步相关的数据结构如下：

```java
ObjectMonitor() {
    _count        = 0; //用来记录该对象被线程获取锁的次数
    _waiters      = 0;
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
}
```

## synchronized的锁的原理

jdk1.6 以后对 synchronized 锁进行了优化，包含偏向锁、轻量级锁、重量级锁; 在了解 synchronized 锁之前，我们需要了解两个重要的概念，一个是对象头、另一个是 monitor

### Java对象头

在 Hotspot 虚拟机中，对象在内存中的布局分为三块区域:对象头、实例数据和对齐填充; Java 对象头是实现 synchronized 的锁对象的基础，一般而言，synchronized 使用的锁对象是存储在 Java 对象头里。它是轻量级锁和偏向锁的关键

### Mawrk Word

Mark Word 用于存储对象自身的运行时数据，如哈希码 (HashCode)、GC分代年龄、锁状态标志、线程持有的 锁、偏向线程 ID、偏向时间戳等等。Java对象头一般占有两个机器码(在32位虚拟机中，1个机器码等于4字节， 也就是32bit)

![](image\20180908110132680.png)

### Monitor

什么是 Monitor?

1.Monitor 是一种用来实现同步的工具

2.与每个 java 对象相关联，即每个 java 对象都有一个 Monitor 与之对应

3.Monitor 是实现 Sychronized(内置锁) 的基础

## synchronized的锁升级和获取过程

首先来了解相关锁的概念：

**自旋锁(CAS)：**让不满足条件的线程等待一会看能不能获得锁，通过占用处理器的时间来避免线程切换带来的开销。自旋等待的时间或次数是有一个限度的，如果自旋超过了定义的时间仍然没有获取到锁，则该线程应该被挂起。

**偏向锁：**大多数情况下，锁总是由同一个线程多次获得。当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，偏向锁是一个可重入的锁。如果锁对象头的Mark Word里存储着指向当前线程的偏向锁，无需重新进行CAS操作来加锁和解锁。当有其他线程尝试竞争偏向锁时，持有偏向锁的线程（不处于活动状态）才会释放锁。偏向锁无法使用自旋锁优化，因为一旦有其他线程申请锁，就破坏了偏向锁的假定进而升级为轻量级锁。

**轻量级锁：**减少无实际竞争情况下，使用重量级锁产生的性能消耗。JVM会现在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中。然后线程会尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针，成功则当前线程获取到锁，失败则表示其他线程竞争锁当前线程则尝试使用自旋的方式获取锁。自旋获取锁失败则锁膨胀会升级为重量级锁。

**重量级锁：**通过对象内部的监视器(monitor)实现，其中monitor的本质是依赖于底层操作系统的Mutex Lock实 现，操作系统实现线程之间的切换需要从用户态到内核态的切换，切换成本非常高。线程竞争不使用自旋，不会消耗CPU。但是线程会进入阻塞等待被其他线程被唤醒，响应时间缓慢。

![](image\20180908110545722.png)

从网上找来的一张图，完美诠释了synchronized锁的升级过程。

### 总结

通过以上可以了解到synchronize的使用、实现原理及锁机制的升级过程。