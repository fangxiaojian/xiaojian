# Java 多线程

## 1、线程创建

### 01、有哪些方法创建线程？

一种，new Thread() 方法创建线程，其他的例如实现接口、继承父类是线程的实现方式

### 02、如何通过 Java 创建进程？

exec 执行cmd 指令。

```java
Runtime runtime = Runtime.getRuntime();
 Process process = runtime.exec("calc"); // 打开计算器
Process process = runtime.exec("cmd /k start http://www.baidu.com"); // 打开百度
process.exitValue();
```

### 03、如何销毁一个线程？

Java中没办法销毁线程。

但是当 Thread.isAlive() 返回 false 时，实际底层的 Thread 已经被销毁了

## 2、线程执行

### 01、如何通过 Java API 启动线程？

thread.start();

方法可以启动线程，执行线程的 run(); 方法

### 02、当有线程 T1、T2 以及 T3， 如何实现 T1 -> T2 -> T3 的执行顺序？

Join 的实现

```java
public static void main(String[] args) throws InterruptedException {
    Thread t1 = new Thread(ThreadExcutionQuesion :: action, "t1");
    Thread t2 = new Thread(ThreadExcutionQuesion :: action, "t2");
    Thread t3 = new Thread(ThreadExcutionQuesion :: action, "t3");

    // start() 仅仅是通知线程启动
//    t1.start();
//    t2.start();
//    t3.start();

    // join 控制线程必须执行完成
//    t1.join();
//    t2.join();
//    t3.join();
    // 这样没办法保证执行顺序的，需要在启动线程时join
    t1.start();
    t1.join();
    t2.start();
    t2.join();
    t3.start();
    t3.join();
  }
 private static void action() {
    System.out.printf("线程[%s] 正在执行...\n", Thread.currentThread().getName());
  }
```

### 03、以上问题请至少提供另外一种实现？

1、 ThreadPoolExecutor 在执行 executor() 方法中会开启线程

2、

```java
private static void threadWait() throws InterruptedException {
    Thread t1 = new Thread(ThreadExcutionQuesion :: action, "t1");
    Thread t2 = new Thread(ThreadExcutionQuesion :: action, "t2");
    Thread t3 = new Thread(ThreadExcutionQuesion :: action, "t3");

    threadStartAndWait(t1);
    threadStartAndWait(t2);
    threadStartAndWait(t3);

}

private static void threadStartAndWait(Thread thread) throws InterruptedException {
    if (Thread.State.NEW.equals(thread.getState())) {
        thread.start();
    }

    while (thread.isAlive()) {
        synchronized (thread) {
            try {
                thread.wait(); // 到底是谁通知 Thread -> thread.notify()
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```



## 3、线程中止

### 01、如何停止一个线程？

```java
// 中断操作（仅仅设置状态，而不是中止线程）
thread.interrupt();
```

### 02、为什么 Java 要放弃 Thread 的 stop() 方法？



### 03、请说明 Thread interrupt()、isInterrupted() 以及 interrupted() 的区别以及意义？

interrupt() 设置状态

isInterrupted() 测试线程是否中断

interrupted() 及判断线程状态又清除



## 4、线程异常

### 01、当线程遇到异常时，到底发生了什么？

线程会中止，但是main线程继续执行

### 02、当线程遇到异常时，如何捕获？

```java
Thread.setDefaultUncaughtExceptionHandler((thread, throwable) -> {
    System.out.printf("线程[%s] 遇到了异常, 详细信息: %s\n", 
                 thread.getName(), 
                 throwable.getMessage());
});
```

### 03、当线程遇到异常时，ThreadPoolExecutor 如何捕获异常？

```java
public static void main(String[] args) throws InterruptedException {

        Thread.setDefaultUncaughtExceptionHandler((thread, throwable) -> {
            System.out.printf("线程[%s] 遇到了异常, 详细信息: %s DefaultUncaughtExceptionHandler\n",
                     thread.getName(), throwable.getMessage());
        });

        ThreadPoolExecutor executorService = new ThreadPoolExecutor(
                1,
                1,
                0,
                TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>()
        ){
            /**
             * 通过覆盖{@link ThreadPoolExecutor#afterExecute(Runnable, Throwable)} 
             * 来达到获取异常的信息
             * @param r
             * @param t
             */
            @Override
            protected void afterExecute(Runnable r, Throwable t) {
                System.out.printf("线程[%s] 遇到了异常, 详细信息: %s afterExecute\n", 
                       Thread.currentThread().getName(), 
                       t.getMessage());
            }
        };

        executorService.execute(() -> {
            throw new RuntimeException("数据达到阈值");
        });

        // 等待1秒钟
        executorService.awaitTermination(1, TimeUnit.SECONDS);

        // 关闭线程池
        executorService.shutdown();
  }
```



## 5、线程状态

### 01、Java 线程有哪些状态，分别代表什么含义？

```java
/**
 * 尚未启动的线程的线程状态。
 * Thread state for a thread which has not yet started.
 */
NEW,

/**
 * 可运行线程的线程状态。处于可运行状态的线程正在Java虚拟机中执
 *  行，但它可能正在等待来自操作系统（例如处理器）的其他资源。
 * Thread state for a runnable thread.  A thread in the runnable
 * state is executing in the Java virtual machine but it may
 * be waiting for other resources from the operating system
 * such as processor.
 */
RUNNABLE,

/**
*  处于阻塞状态的线程的线程状态等待监视器锁定。处于阻塞状态的线程正在
* 等待监视器锁定，以在调用{@link Object＃wait（）对象之后输入同步
* 块/方法或重新输入同步块/方法。等待}。
 * Thread state for a thread blocked waiting for a monitor lock.
 * A thread in the blocked state is waiting for a monitor lock
 * to enter a synchronized block/method or
 * reenter a synchronized block/method after calling
 * {@link Object#wait() Object.wait}.
 */
BLOCKED,

/**
* 等待线程的线程状态。
* 由于调用以下方法之一，线程处于等待状态：
 * Thread state for a waiting thread.
 * A thread is in the waiting state due to calling one of the
 * following methods:
 * <ul>
 *   <li>{@link Object#wait() Object.wait} with no timeout</li>
 *   <li>{@link #join() Thread.join} with no timeout</li>
 *   <li>{@link LockSupport#park() LockSupport.park}</li>
 * </ul>
 *
 * 处于等待状态的线程正在等待另一个线程执行特定操作。
 * <p>A thread in the waiting state is waiting for another thread to
 * perform a particular action.
 *
 * For example, a thread that has called <tt>Object.wait()</tt>
 * on an object is waiting for another thread to call
 * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
 * that object. A thread that has called <tt>Thread.join()</tt>
 * is waiting for a specified thread to terminate.
 */
WAITING,

/**
* 具有指定等待时间的等待线程的线程状态。
* 由于以指定的正等待时间调用以下方法之一，因此线程处于定时等待状态：
 * Thread state for a waiting thread with a specified waiting time.
 * A thread is in the timed waiting state due to calling one of
 * the following methods with a specified positive waiting time:
 * <ul>
 *   <li>{@link #sleep Thread.sleep}</li>
 *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
 *   <li>{@link #join(long) Thread.join} with timeout</li>
 *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
 *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
 * </ul>
 */
TIMED_WAITING,

/**
* 终止线程的线程状态。
* 线程已完成执行。
 * Thread state for a terminated thread.
 * The thread has completed execution.
 */
TERMINATED;
```

### 02、如何获取当前 JVM 所有的线程状态？

```java
ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
long[] threadIds = threadMXBean.getAllThreadIds();

for (long threadId : threadIds) {
    ThreadInfo threadInfo = threadMXBean.getThreadInfo(threadId);
    System.out.println(threadInfo.toString());
}
```

> 输出结果：
>
> "Monitor Ctrl-Break" Id=6 RUNNABLE (in native)
>
>
> "Attach Listener" Id=5 RUNNABLE
>
>
> "Signal Dispatcher" Id=4 RUNNABLE
>
>
> "Finalizer" Id=3 WAITING on java.lang.ref.ReferenceQueue$Lock@1b6d3586
>
>
> "Reference Handler" Id=2 WAITING on java.lang.ref.Reference$Lock@4554617c
>
>
> "main" Id=1 RUNNABLE

### 03、如何获取线程的资源消费情况？

```java
ThreadMXBean threadMXBean = (ThreadMXBean) ManagementFactory.getThreadMXBean();
long[] threadIds = threadMXBean.getAllThreadIds();

for (long threadId : threadIds) {
    long bytes = threadMXBean.getThreadAllocatedBytes(threadId);
    long mbytes = bytes / 1024;
    System.out.printf("线程[ID: %d] 分配内存:  %s  KB\n", threadId, mbytes);
}
```

> 输出结果：
>
> 线程[ID: 6] 分配内存:  289  KB
> 线程[ID: 5] 分配内存:  0  KB
> 线程[ID: 4] 分配内存:  0  KB
> 线程[ID: 3] 分配内存:  0  KB
> 线程[ID: 2] 分配内存:  0  KB
> 线程[ID: 1] 分配内存:  2082  KB



## 6、线程同步

### 01、请说明 synchronized 关键字在修饰方法与代码块中的区别？

主要是字节码的区别

修饰方法 的字节码  ACC_SYNCHRONIZED

```java
private static synchronized void synchronizedMethod();
  descriptor: ()V
  flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNCHRONIZED
  Code:
    stack=0, locals=0, args_size=0
       0: return
    LineNumberTable:
      line 22: 0
```

修饰代码块 的字节码

```java
private static void synchronizedBlock();
  descriptor: ()V
  flags: ACC_PRIVATE, ACC_STATIC
  Code:
    stack=2, locals=2, args_size=0
       0: ldc           #2 
       2: dup
       3: astore_0
       4: monitorenter  //  进入
       5: aload_0
       6: monitorexit   // 退出
       7: goto          15
      10: astore_1
      11: aload_0
      12: monitorexit
      13: aload_1
      14: athrow
      15: return
    Exception table:
       from    to  target type
           5     7    10   any
          10    13    10   any
    LineNumberTable:
      line 15: 0
      line 17: 5
      line 18: 15
    StackMapTable: number_of_entries = 2
      frame_type = 255 /* full_frame */
        offset_delta = 10
        locals = [ class java/lang/Object ]
        stack = [ class java/lang/Throwable ]
      frame_type = 250 /* chop */
        offset_delta = 4
```



### 02、请说明 synchronized 关键字与 ReentrantLock 之间的区别？



### 03、请解释偏向锁对 synchronized 与 ReentrantLock 的价值？

偏向锁只对 synchronized 有用



## 7、线程通信

### 01、为什么 wait() 和 notify() 以及 notifyAll() 方法属于 Object，并解释他们的作用？

1. 为什么 wait() 和 notify() 以及 notifyAll() 方法属于 Object
   - monitor -> Object   // 监视对象

### 02、为什么 Object wait() 和 notify() 以及 notifyAll() 方法必须 synchronized 之中执行？

object#wait() 获取锁的对象，释放锁，当前线程又被阻塞。

LockSupport#park();    与wait 相似， 但是 wait 会被JVM 主动唤醒， park不会，所以容易造成死锁。



### 03、请通过 Java 代码模拟实现 wait() 和 notify() 以及 notifyAll() 的语义？



## 8、线程退出

### 01、当主线程退出时，守候线程会执行完毕吗？

当主线程退出时，子线程不一定执行。

有可能守护线程执行完主线程才退出。

守护线程的执行依赖于执行时间（非唯一评判）

### 02、请说明 Sthutdown Hook 线程的使用场景，以及如何触发执行？

#### Shutdown Hook 描述

> `Shutdown hook`是一个initialized but unstarted thread。当JVM开始执行shutdown sequence时，会并发运行所有registered `Shutdown Hook`。这时，在`Shutdown Hook`这个线程里定义的操作便会开始执行。

> 需要注意的是，在`Shutdown Hook`里执行的操作应当是不太耗时的。因为在用户注销或者操作系统关机导致的JVM shutdown的例子中，系统只会预留有限的时间给未完成的工作，超时之后还是会强制关闭。

#### Shutdown Hook 什么时候会被调用

- 程序正常停止
  - Reach the end of program
  - System.exit
- 程序异常退出
  - NPE
  - OutOfMemory
- 受到外界影响停止
  - Ctrl + C
  - 用户注销或者关机

#### Shutdown Hook 如何使用

```java
public static void main(String[] args) {

    Runtime runtime = Runtime.getRuntime();

    runtime.addShutdownHook(new Thread((ShutdownHookQuestion::action), 
                                       "Shutdown Hook Question"));
}

private static void action() {
    System.out.printf("线程[%s] 正在执行...\n", 
                      Thread.currentThread().getName());
}
```

### 03、如何确保在主线程退出前，所有线程执行完毕？

使用线程组

```java
public static void main(String[] args) throws InterruptedException {

    // main 线程 -> 子线程
    Thread t1 = new Thread(CompleteAllThreadsQuestion::action, "t1");
    Thread t2 = new Thread(CompleteAllThreadsQuestion::action, "t2");
    Thread t3 = new Thread(CompleteAllThreadsQuestion::action, "t3");
    Thread t4 = new Thread(CompleteAllThreadsQuestion::action, "t4");
    Thread t5 = new Thread(CompleteAllThreadsQuestion::action, "t5");

    // 不确定 t1、t2、t3、t4... 是否调用 start()
    t1.start();
    t2.start();
    t3.start();
    t4.start();
    t5.start();

    // 创建了 N 个 Thread

    Thread mainThread = Thread.currentThread();

    // 获取 main 线程组
    ThreadGroup threadGroup = mainThread.getThreadGroup();

    // 获取活跃的线程数
    int count = threadGroup.activeCount();
    Thread[] threads = new Thread[count];

    // 把所有的线程复制到 threads 数组
    threadGroup.enumerate(threads, true);

    for (Thread thread : threads) {
        System.out.printf("线程[%s] 正在活跃中...\n", thread.getName());
    }
}

private static void action() {
    System.out.printf("线程[%s] 正在执行...\n", 
                      Thread.currentThread().getName());
}
```



# Java 并发集合框架

## 1、线程安全集合

### 01、请在 Java 集合框架以及 J.U.C 框架中各举出 List、Set 以及 Map 的实现？



### 02、如何将普通 List、Set 以及 Map 转化为线程安全对象？



### 03、如何在 Java 9+ 实现以上问题？



## 2、线程安全 LIST

### 01、请说明 List、Vector 以及 CopyOnWriteArrayList 的相同点和不同点？

1. Vector 是List 的实现， 他实现了 List 接口， 全部加锁
2. List 是线程不安全的
3. CopyOnWriteArrayList 读的时候不加锁

### 02、请说明 Collections#synchronizedList(list) 与 Vector 的相同点和不同点？

1. 都是通过 synchronized 关键字的方式进行实现
2. 返回类型不同，Collections#synchronizedList(list) 返回 list， 而 Vector 返回 Vector

### 03、Arrays#asList(Object...) 方法是线程安全的吗？如果不是的话，如何实现替代方案？

不是线程安全的。

```java
// Java < 5, Collections#synchronizedList
// Java > 5, CopyOnWriteArrayList
// List.of  Java 9+
```



## 3、线程安全 SET

### 01、请至少举出三种线程安全的 Set 实现？

SynchronizedSet

CopyOnWriteArraySet

ConcurrentSkipSet

### 02、在 J.U.C 框架中，存在 HashSet 的线程安全实现？如果不存在的话，要如何实现？

不存在

参考 HashSet

HashSet 里面是使用 HashMap 进行操作的

J.U.C 中可以使用 ConcurrentHashMap 对 HashSet 进行线程安全操作。

### 03、当 Set#iterator() 方法返回 Iterator 对象后，能否在其迭代中，给 Set 对象添加新的元素？

不一定，



## 4、线程安全 MAP

### 01、 请说明 Hashtable、HaspMap 以及 ConcurrentHashMap 的区别？

Hashtable -> key、vlaue 都不能为空

ConcurrentHashMap -> key、vlaue 都不能为空

HashMap -> key、value都能为空



### 02、请说明 ConcurrentHashMap 在不同 JDK 中的实现？

jdk6 读 部分锁  写  完全锁

jdk7 读  不锁     写  需要锁

jdk8 读  不锁     写  需要锁    加入 红黑树

### 03、请说明 ConcurrentHashMap 与 ConcurrentSkipListMap 各自的优势与不足？

ConcurrentHashMap   写的时候加锁   --  

ConcurrentSkipListMap  写的时候不加锁  --  通过空间换时间的方式，使速度加快