# 开篇语

我们在 《打动面试官：线程池流程编排中的运用实战》一文中将流程引擎简单地完善了一下，本文在其基础上继续进行改造，建议同学可以先看看 GitHub 上的代码，或者看看之前的文章。

# 1 回顾

流程引擎编排的对象，我们称为组件（就是 SpringBean），之前我们给组件定义了通用的接口，组件实现时就实现这个接口，代码如下：

![](..\image\44-1.png)

我们定义了 DomainAbilityBean 接口，入参和出参都是 FlowContent，FlowContent 我们称为上下文。

# 2 ThreadLocal 实现

上下文传参除了 FlowContent 实现外，ThreadLocal 也是可以实现的，我们来演示一下：

## 2.1 定义 ThreadLocal 上下文工具类

首先我们使用 ThreadLocal 定义了上下文工具类，并且定义了 put、get 方法，方便使用，代码如下：

```java
public class ContextCache implements Serializable {
    private static final long serialVersionUID = 2136539028591849277L;
    // 使用 ThreadLocal 缓存上下文信息
    public static final ThreadLocal<Map<String,String>> CACHE = new ThreadLocal<>();
    /**
    * 放数据
    * @param sourceKey
    */
    public static final void putAttribute(String sourceKey,String value){
        Map<String,String> cacheMap = CACHE.get();
        if(null == cacheMap){
        	cacheMap = new HashMap<>();
        }
        cacheMap.put(sourceKey,value);
        CACHE.set(cacheMap);
    }
    /**
    * 拿数据
    * @param sourceKey
    */
    public static final String getAttribute(String sourceKey){
        Map<String,String> cacheMap = CACHE.get();
        if(null == cacheMap){
            return null;
        }
        return cacheMap.get(sourceKey);
    }
}
```

如果你想往 ThreadLocal 放数据，调用 ContextCache.putAttribute 方法，如果想从 ThreadLocal 拿数据，调用 ContextCache.getAttribute 方法即可。

我们写了两个组件，一个组件放数据，一个组件拿数据，如下：

![](..\image\44-2.png)

我们把两个 SpringBean 注册到流程注册中心中，让其按照先执行 BeanThree 再执行 BeanFive 的顺序进行执行，运行 DemoApplication 类的 main 方法进行执行，执行结果如下：

![](..\image\44-3.png)

从打印的日志可以看到，在 Spring 容器管理的 SpringBean 中，ThreadLocal 也是可以储存中间缓存值的。

# 3 开启子线程

我们做一个实验，我们在 BeanFive 中开启子线程，然后再从 ThreadLocal 中拿值，看看能否拿到值，BeanFive 的代码修改成如下：

![](..\image\44-4.png)

我们再来运行一下，打印的日志如下：

![](..\image\44-5.png)

从打印的日志中，我们发现在子线程中从 ThreadLocal 取值时，并没有取得值，这个原因主要是我们之前说的，线程在创建的时候，并不会把父线程的 ThreadLocal 中的值拷贝给子线程的 ThreadLocal，解决方案就是把 ThreadLocal 修改成 InheritableThreadLocal，代码修改如下：

![](..\image\44-6.png)

我们再次运行，结果如下：

![](..\image\44-7.png)

从运行结果看，我们成功的在子线程中拿到值。

# 4 线程池 + ThreadLocal

如果是拿数据的 springBean 是丢给线程池执行的，我们能够成功的从 ThreadLocal 中拿到数据么？

首先我们在放数据的 springBean 中，把放的值修改成随机的，接着拿数据的 SpringBean 修改成异步执行，代码修改如下：

![](..\image\44-8.png)

为了能快速看到效果，我们把线程池的 coreSize 和 maxSize 全部修改成 3，并让任务沉睡一段时间，这样三个线程肯定消费不完任务，大量任务都会到队列中去排队，我们修改一下测试脚本，如下：

![](..\image\44-9.png)

我们期望的结果：
1. 线程池中执行的 BeanFive 可以成功从 ThreadLocal 中拿到数据；
2. 能够从 ThreadLocal 拿到正确的数据，比如 BeanThree 刚放进 key1，value5，那么期望在 BeanFive 中根据 key1 能拿出 value5，而不是其它值。

我们运行一下，结果如下：

![](..\image\44-10.png)

从结果中可以看到，并没有符合我们的预期，我们往 ThreadLocal 中 put 进很多值，但最后拿出来的值却很多都是 value379，都为最后 put 到 ThreadLocal 中的值。

这个原因主要是 ThreadLocal 存储的 HashMap 的引用都是同一个，main 主线程可以修改 HashMap 中的值，子线程从 ThreadLocal 中拿值时，也是从 HashMap 中拿值，从而导致不能把 put 的值通过 ThreadLocal 正确的传递给子线程。

为了证明是这个原因，我们在从 ThreadLocal 放、拿值的地方，把 HashMap 的内存地址都打印出来，改动代码如下：

![](..\image\44-11.png)

我们再次运行测试代码，运行的结果如下：

![](..\image\44-12.png)

从测试结果中可以看到，不管是主线程还是子线程和 ThreadLocal 进行交互时，HashMap 都是同一个，也就是说 ThreadLocal 中保存的 HashMap 是共享的，这就导致了线程安全的问题，子线程读取到的值就会混乱掉。

# 5 解决方案

针对这个问题，我们提出了一种解决方案，在把任务提交到线程池时，我们进行 HashMap 的拷贝，这样子线程的 HashMap 和 main 线程的 HashMap 就不同了，可以解决上面的问题。

我们提交任务时， 使用的是 Runnable，要实现 HashMap 的拷贝的话，我们需要把 Runnable 进行一层包装，包装的代码如下：

![](..\image\44-13.png)

运行结果如下:

![](..\image\44-14.png)

从运行结果中可以看出，线程池拿出来的 value 已经是正确的了。

# 6 总结

本文通过 ThreadLocal 来改造流程引擎中的上下文传递，希望能够加深大家对 ThreadLocal 的认识和使用技巧，有兴趣的同学可以把我们的代码下载下来，跑跑看。