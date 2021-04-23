# 引导语

上两小节我们学习了 AQS，本章我们就要来学习一下第一个 AQS 的实现类：ReentrantLock，看看其底层是如何组合 AQS ，实现了自己的那些功能。

本章的描述思路是先描述清楚 ReentrantLock 的构成组件，然后使用加锁和释放锁的方法把这些组件串起来。

# 1 类注释

ReentrantLock 中文我们习惯叫做可重入互斥锁，可重入的意思是同一个线程可以对同一个共享资源重复的加锁或释放锁，互斥就是 AQS 中的排它锁的意思，只允许一个线程获得锁。

我们来一起来看下类注释上都有哪些重要信息：

1. 可重入互斥锁，和 synchronized 锁具有同样的功能语义，但更有扩展性；
2. 构造器接受 fairness 的参数，fairness 是 ture 时，保证获得锁时的顺序，false 不保证；
3. 公平锁的吞吐量较低，获得锁的公平性不能代表线程调度的公平性；
4. tryLock() 无参方法没有遵循公平性，是非公平的（lock 和 unlock 都有公平和非公平，而 tryLock 只有公平锁，所以单独拿出来说一说）。

我们补充一下第二点，ReentrantLock 的公平和非公平，是针对获得锁来说的，如果是公平的，可以保证同步队列中的线程从头到尾的顺序依次获得锁，非公平的就无法保证，在释放锁的过程中，我们是没有公平和非公平的说法的。

# 2 类结构

ReentrantLock 类本身是不继承 AQS 的，实现了 Lock 接口，如下：

```java
public class ReentrantLock implements Lock, java.io.Serializable {}
```

Lock 接口定义了各种加锁，释放锁的方法，接口有如下几个：

```java
// 获得锁方法，获取不到锁的线程会到同步队列中阻塞排队
void lock();
// 获取可中断的锁
void lockInterruptibly() throws InterruptedException;
// 尝试获得锁，如果锁空闲，立马返回 true，否则返回 false
boolean tryLock();
// 带有超时等待时间的锁，如果超时时间到了，仍然没有获得锁，返回 false
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
// 释放锁
void unlock();
// 得到新的 Condition
Condition newCondition();
```

ReentrantLock 就负责实现这些接口，我们使用时，直接面对的也是这些方法，这些方法的底层实现都是交给 Sync 内部类去实现的，Sync 类的定义如下：

```java
abstract static class Sync extends AbstractQueuedSynchronizer {}
```

Sync 继承了 AbstractQueuedSynchronizer ，所以 Sync 就具有了锁的框架，根据 AQS 的框架，Sync 只需要实现 AQS 预留的几个方法即可，但 Sync 也只是实现了部分方法，还有一些交给子类 NonfairSync 和 FairSync 去实现了，NonfairSync 是非公平锁，FairSync 是公平锁，定义如下：

```java
// 同步器 Sync 的两个子类锁
static final class FairSync extends Sync {}
static final class NonfairSync extends Sync {}
```

几个类整体的结构如下：

![](..\image\ReentrantLock.png)

图中 Sync、NonfairSync、FairSync 都是静态内部类的方式实现的，这个也符合 AQS 框架定义的实现标准。

流程

![](..\image\ReentrantLock-lock.png)



# 3 构造器

ReentrantLock 构造器有两种，代码如下：

```java
// 无参数构造器，相当于 ReentrantLock(false)，默认是非公平的
public ReentrantLock() {
	sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

无参构造器默认构造是非公平的锁，有参构造器可以选择。

从构造器中可以看出，公平锁是依靠 FairSync 实现的，非公平锁是依靠 NonfairSync 实现的。

# 4 Sync 同步器

Sync 表示同步器，继承了 AQS，UML 图如下：

![](..\image\Sync.png)

从 UML 图中可以看出，lock 方法是个抽象方法，留给 FairSync 和 NonfairSync 两个子类去实现，我们一起来看下剩余重要的几个方法。

## 4.1 nonfairTryAcquire

```java
// 尝试获得非公平锁
final boolean nonfairTryAcquire(int acquires) {
	final Thread current = Thread.currentThread();
	int c = getState();
	// 同步器的状态是 0，表示同步器的锁没有人持有
	if (c == 0) {
		// 当前线程持有锁
		if (compareAndSetState(0, acquires)) {
			// 标记当前持有锁的线程是谁
			setExclusiveOwnerThread(current);
			return true;
		}
	}
	// 如果当前线程已经持有锁了，同一个线程可以对同一个资源重复加锁，代码实现的是可重入锁
	else if (current == getExclusiveOwnerThread()) {
		// 当前线程持有锁的数量 + acquires
		int nextc = c + acquires;
		// int 是有最大值的，<0 表示持有锁的数量超过了 int 的最大值
		if (nextc < 0) // overflow
			throw new Error("Maximum lock count exceeded");
		setState(nextc);
		return true;
	}
	//否则线程进入同步队列
	return false;
}
```

以上代码有三点需要注意：
1. 通过判断 AQS 的 state 的状态来决定是否可以获得锁，0 表示锁是空闲的；
2. else if 的代码体现了可重入加锁，同一个线程对共享资源重入加锁，底层实现就是把 state + 1，并且可重入的次数是有限制的，为 Integer 的最大值；
3. 这个方法是非公平的，所以只有非公平锁才会用到，公平锁是另外的实现。

无参的 tryLock 方法调用的就是此方法，tryLock 的方法源码如下：

```java
public boolean tryLock() {
	// 入参数是 1 表示尝试获得一次锁
	return sync.nonfairTryAcquire(1);
}
```

## 4.2 tryRelease

```java
// 释放锁方法，非公平和公平锁都使用
protected final boolean tryRelease(int releases) {
	// 当前同步器的状态减去释放的个数，releases 一般为 1
	int c = getState() - releases;
	// 当前线程根本都不持有锁，报错
	if (Thread.currentThread() != getExclusiveOwnerThread())
		throw new IllegalMonitorStateException();
	boolean free = false;
	// 如果 c 为 0，表示当前线程持有的锁都释放了
	if (c == 0) {
		free = true;
		setExclusiveOwnerThread(null);
	}
	// 如果 c 不为 0，那么就是可重入锁，并且锁没有释放完，用 state 减去 releases 即可，无需做其
	setState(c);
	return free;
}
```

tryRelease 方法是公平锁和非公平锁都公用的，在锁释放的时候，是没有公平和非公平的说法的。

从代码中可以看到，锁最终被释放的标椎是 state 的状态为 0，在重入加锁的情况下，需要重入解锁相应的次数后，才能最终把锁释放，比如线程 A 对共享资源 B 重入加锁 5 次，那么释放锁的话，也需要释放 5 次之后，才算真正的释放该共享资源了。

# 5 FairSync 公平锁

FairSync 公平锁只实现了 lock 和 tryAcquire 两个方法，lock 方法非常简单，如下：

```java
// acquire 是 AQS 的方法，表示先尝试获得锁，失败之后进入同步队列阻塞等待
final void lock() {
	acquire(1);
}
```

tryAcquire 方法是 AQS 在 acquire 方法中留给子类实现的抽象方法，FairSync 中实现的源码如下：

```java
protected final boolean tryAcquire(int acquires) {
    //获取当前线程
    final Thread current = Thread.currentThread();
    //获取lock对象的上锁状态，如果锁是自由状态则=0，如果被上锁则为1，大于1表示重入
    int c = getState();
    if (c == 0) { //没人占用锁--->我要去上锁----1、锁是自由状态
        // hasQueuedPredecessors 是实现公平的关键
        // 会判断当前线程是不是属于同步队列的头节点的下一个节点(头节点是释放锁的节点)
        // 如果是(返回false)，符合先进先出的原则，可以获得锁
        // 如果不是(返回true)，则继续等待
        if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
        	//设置当前线程为拥有锁的线程，方面后面判断是不是重入（只需把这个线程拿出来判断是否当前线程即可判断重入）    
            setExclusiveOwnerThread(current);
        	return true;
        }
    }
    // 可重入锁
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
        	throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

代码和 Sync 的 nonfairTryAcquire 方法实现类似，唯一不同的是在获得锁时使用 hasQueuedPredecessors 方法体现了其公平性。

>  hasQueuedPredecessors 

hasQueuedPredecessors 判断是否需要排队的源码分析，这里需要记住一点，整个方法如果最后返回false，则去加锁，如果返回true则不加锁，因为这个方法被取反了

```java
public final boolean hasQueuedPredecessors() {
    Node t = tail; 
    Node h = head;
    Node s;
    /**
     * 下面提到的所有不需要排队，并不是字面意义的不需要排队，我实在想不出什么词语来描述这个“不需要排队”；
     * 
     * 不需要排队有两种情况
     * 一：队列没有初始化，不需要排队，不需要排队，不需要排队；直接去加锁，但是可能会失败；为什么会失败呢？
     * 假设两个线程同时来lock，都看到队列没有初始化，都认为不需要排队，都去进行CAS修改计数器；但是某个线程t1先拿到锁，那么另外一个t2则会CAS失败，这个时候他就会初始化队列，并排队
     *
     * 二：队列被初始化了，但是tc过来加锁，发觉队列当中第一个排队的就是自己（比如重入；那么什么是第一个排队的呢？下面解释了，往下看）这个时候他也-不需要排队，不需要排队，不需要排队；为什么不需要排对？
    * 因为队列当中第一个排队的线程他回去尝试获取一下锁，因为有可能这个时候持有锁锁的那个线程可能释放了锁，如果释放了就直接获取锁执行。但是如果没有释放他就会去排队，所以这里的不需要排队，不是真的不需要排队，好好理解一下
     *
     * h != t 判断首不等于尾这里要分三种情况
     * 1、队列没有初始化，也就是第一个线程tf来加锁的时候那么这个时候队列没有初始化，h和t都是null，那么这个时候不等于不成立（false）那么由于是&&运算后面的就不会走了，直接返回false表示不需要排队，而前面又是取反（if (!hasQueuedPredecessors()），所以会直接去cas加锁。
     * ----------第一种情况总结：队列没有初始化没人排队，那么我直接不排队，直接上锁；合情合理、有理有据令人信服；好比你去买票，服务员都闲的蛋疼，没人排队，你直接过去价钱拿票
     *
    * 2、队列被初始化了，后面我们会分析队列初始化的流程，如果队列被初始化那么h!=t则成立；h != t 返回true；但是是&&运算，故而还需要进行后续的判断
    * （有人可能会疑问，比如队列里面只有一个数据，那么头和尾都是同一个怎么会成立呢？其实这是第三种情况--队列里面只有一个数据；这里先不考虑，假设现在队列里面有大于1个数据）
    * 大于1个数据则成立;继续判断把h.next赋值给s；s有是头的下一个，则表示他是队列当中参与排队的线程而且是排在最前面的；为什么是s最前面不是h嘛？诚然h是队列里面的第一个，但是不是排队的第一个；
    * 因为h是持有锁的，但是不参与排队；这个也很好理解，比如你去买火车票，你如果是第一个这个时候售票员已经在给你服务了，你不算排队，你后面的才算排队；
    * 队列里面的h是不参与排队的这点一定要明白参考下面关于队列初始化的解释---因为h要么是虚拟出来的节点，要么是持有锁的节点；什么时候是虚拟的呢？什么时候是持有锁的节点呢？下文分析
    * 然后判断s是否等于空，其实就是判断队列里面是否只有一个数据；假设队列大于1个，那么肯定不成立（s==null---->false），因为大于一个h.next肯定不为空；
    * 由于是||运算如果返回false，还要判断s.thread != Thread.currentThread()；这里又分为两种情况
    *        2.1 s.thread != Thread.currentThread() 返回true，就是当前线程不等于在排队的第一个线程s；
    *              那么这个时候整体结果就是h!=t：true; （s==null false || s.thread != Thread.currentThread() true------> 最后true）结果： true && true 方法最终放回true，那么去则需要去排队
    *              其实这样符合情理，队列不为空，有人在排队，而且第一个排队的人和现在来参与竞争的人不是同一个，那么你就乖乖去排队
    *        2.2 s.thread != Thread.currentThread() 返回false 表示当前来参与竞争锁的线程和第一个排队的线程是同一个线程
    *            那么这个时候整体结果就是h!=t：true; （s==null false || s.thread != Thread.currentThread() false------> 最后false）结果 true && false 方法最终放回false，那么去则不需要去排队
    *            不需要排队则调用 compareAndSetState(0, acquires) 去改变计数器尝试上锁；这里又分为两种情况（日了狗了这一行代码；有同学课后反应说子路老师老师老是说这个AQS难，你现在仔细看看这一行代码的意义，真的不简单的）
    *             2.2.1 第一种情况加锁成功？有人会问为什么会成功啊，很简单假如这个时候h也就是持有锁的那个线程执行完了，释放锁了，那么肯定成功啊；成功则执行 setExclusiveOwnerThread(current); 然后返回true 自己看代码
    *             2.2.2 第二种情况加锁失败？有人会问为什么会失败啊。很简单假如这个时候h也就是持有锁的那个线程没执行完，没释放锁，那么肯定失败啊；失败则直接返回false，不会进else if（java基础，else if是相对于 if (c == 0)的）
    *                   那么如果失败怎么办呢？后面分析；
    *
    *----------第二种情况总结，如果队列被初始化了，而且至少有一个人在排队那么自己也去排队；但是有个插曲；他会去看看那个第一个排队的人是不是自己，如果是自己那么他就去尝试假设；尝试看看锁有没有释放
    *----------也合情合理，好比你去买票，如果有人排队，那么你乖乖排队，但是你会去看看第一个排队的人是不是你女朋友；或者男朋友
    *----------如果是你女朋友就相当于是你自己（这里实在想不出现实世界关于重入的例子，只能用男女朋友来替代），你就叫你女朋友看看售票员有没有搞完，有没有轮到你女朋友，因为你女朋友是第一个排队的
    *----------疑问：比如如果在在排队，那么他是park状态，如果是park状态，自己怎么还可能重入啊。希望有同学可以想出来为什么和我讨论一下，作为一个菜逼，希望有人教教我
    *          
    * 3、队列被初始化了，但是里面只有一个数据；什么情况下才会出现这种情况呢？可能有人会说ts加锁的时候里面就只有一个数据；其实不是，因为队列初始化的时候会虚拟一个h作为头结点，当前线程作为第一个排队的节点
    * 为什么这么做呢？因为aqs认为h永远是不排队的，假设你不虚拟节点出来那么ts就是h，而ts其实需要排队的，因为这个时候tf可能没有执行完，ts得不到锁，故而他需要排队；
    * 那么为什么要虚拟为什么ts不直接排在tf之后呢，上面已经时说明白了，tf来上锁的时候队列都没有，他不进队列，故而ts无法排在tf之后，只能虚拟一个null节点出来；
    * 那么问题来了；究竟上面时候才会出现队列当中只有一个数据呢？假设原先队列里面有5个人在排队，当前面4个都执行完了，轮到第五个线程得到锁的时候；他会把自己设置成为头部，而尾部又没有，故而队列当中只有一个h就是第五个
    * 至于为什么需要把自己设置成头部；其实已经解释了，因为这个时候五个线程已经不排队了，他拿到锁了，所以他不参与排队，故而需要设置成为h；即头部；所以这个时间内，队列当中只有一个节点
    * 关于加锁成功后把自己设置成为头部的源码，后面会解析到；继续第三种情况的代码分析，记得这个时候队列已经初始化了，但是只有一个数据，并且这个数据所代表的线程是持有锁
    * h != t false 由于后面是&&运算，故而返回false可以不参与运算，整个方法返回false；不需要排队
    *
    *
    *-------------第三种情况总结：如果队列当中只有一个节点，而这种情况我们分析了，这个节点就是当前持有锁的那个节点，故而我不需要排队，进行cas；
    *-------------如果持有锁的线程释放了锁，那么我能成功上锁
    *-------------
    *
    **/
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```



# 6 NonfairSync 非公平锁

NonfairSync 底层实现了 lock 和 tryAcquire 两个方法，如下:

```java
// 加锁
final void lock() {
	// cas 给 state 赋值
	if (compareAndSetState(0, 1))
		// cas 赋值成功，代表拿到当前锁，记录拿到锁的线程
		setExclusiveOwnerThread(Thread.currentThread());
	else
		// acquire 是抽象类AQS的方法,
		// 会再次尝试获得锁，失败会进入到同步队列中
		acquire(1);
}

// 直接使用的是 Sync.nonfairTryAcquire 方法
protected final boolean tryAcquire(int acquires) {
	return nonfairTryAcquire(acquires);
}
```

# 7 如何串起来

以上内容主要说了 ReentrantLock 的基本结构，比较零散，那么这些零散的结构如何串联起来呢？我们是通过 lock、tryLock、unlock 这三个 API 将以上几个类串联起来，我们来一一看下。

## 7.1 lock 加锁

lock 的代码实现：

```java
public void lock() {
	sync.lock();
}
```

其底层的调用关系(只是简单表明调用关系，并不是完整分支图)如下：

![](..\image\FairSync-NonfairSync.png)

## 7.2 tryLock 尝试加锁

tryLock 有两个方法，一种是无参的，一种提供了超时时间的入参，两种内部是不同的实现机制，代码分别如下:

```java
// 无参构造器
public boolean tryLock() {
	return sync.nonfairTryAcquire(1);
}
// timeout 为超时的时间，在时间内，仍没有得到锁，会返回 false
public boolean tryLock(long timeout, TimeUnit unit)
		throws InterruptedException {
	return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

接着我们一起看下两个 tryLock 的调用关系图，下图显示的是无参 tryLock 的调用关系图，如下：

![](..\image\NonfairSync-tryLock.png)

我们需要注意的是 tryLock 无参方法底层走的就是非公平锁实现，没有公平锁的实现。下图展示的是带有超时时间的有参 tryLock 的调用实现图：

![](..\image\FairSync-NonfairSync-tryLock.png)

## 7.3 unlock 释放锁

unlock 释放锁的方法，底层调用的是 Sync 同步器的 release 方法，release 是 AQS 的方法，分成两步：

1. 尝试释放锁，如果释放失败，直接返回 false；
2. 释放成功，从同步队列的头节点的下一个节点开始唤醒，让其去竞争锁。

第一步就是我们上文中 Sync 的 tryRelease 方法（4.1），第二步 AQS 已经实现了。

unLock 的源码如下：

```java
// 释放锁
public void unlock() {
	sync.release(1);
}
```

## 7.4 Condition

ReentrantLock 对 Condition 并没有改造，直接使用 AQS 的 ConditionObject 即可。

# 8 总结

这就是我们在研究完 AQS 源码之后，碰到的第一个锁，是不是感觉很简单，AQS 搭建了整个锁架构，子类锁只需要根据场景，实现 AQS 对应的方法即可，不仅仅是 ReentrantLock 是这样，JUC 中的其它锁也都是这样，只要对 AQS 了如指掌，锁其实非常简单。