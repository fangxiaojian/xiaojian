# Netty 实战

## 一、Netty 的概念及体系结构

### 1、Netty -- 异步和事件驱动

> - Java网络编程
> - Netty简介
> - Netty的核心组件

​        Netty是一款异步的事件驱动的 **网络** 应用程序框架，支持快速地开发可维护的 **高性能** 的面向协议的服务器和客户端。

#### Netty 的核心组件

- Channel；
- 回调；
- Future；
- 事件和 ChannelHandler

#### Channel

​        Channel 是Java NIO 的一个基本构造。

> ​        它代表一个到实体（如一个硬件设备、一个文件、一个网络套接字或者一个能够执行一个或者多个不同的I/O操作的程序组件）的开放连接，如读操作和写操作。

​        目前，可以把 `Channel` 看作是传入（入站）或者传出（出站）数据的载体。因此，它可以被打开或者被关闭，连接或者断开连接。

#### 回调

​        一个回调其实就是一个方法，一个指向已经被提供给另外一个方法的方法的引用。这使得后者（指接受回调的方法。）可以在适当的时候调用前者。回调在广泛的编程场景中都有应用，而且也是在操作完成后通知相关方最常见的方式之一。

​        Netty在内部使用了回调来处理事件；当一个回调被触发时，相关的事件可以被一个 `interface-ChannelHandler` 的实现处理。代码清单1-2展示了一个例子：当一个新的连接已经被创建时， `ChannelHandler` 的 `channelActive()` 回调方法将会被调用，并将打印出一条信息。  

#### Future

​        `Future` 提供了另一种在操作完成时通知应用程序的方式。这个对象可以看作是一个异步操作的结果的占位符；它将在未来的某个时刻完成，并提供对其结果的访问。

​        Netty 提供了它自己的实现——`ChannelFuture`，用于在执行异步操作的时候使用。

​        `ChannelFuture`提供了几种额外的方法，这些方法使得我们能够注册一个或者多个`ChannelFutureListener` 实例。监听器的回调方法 `operationComplete()`，将会在对应的操作完成时被调用(如果在 `ChannelFutureListener` 添加到`ChannelFuture`的时候，`ChannelFuture`已经完成，那么该`ChannelFutureListener`将会被直接地通知)。然后监听器可以判断该操作是成功地完成了还是出错了。如果是后者，我们可以检索产生的`Throwable`。简而言之，由`ChannelFutureListener`提供的通知机制消除了手动检查对应的操作是否完成的必要。

​        每个Netty的出站I/O操作都将返回一个`ChannelFuture`；也就是说，它们都不会阻塞。正如我们前面所提到过的一样，Netty完全是异步和事件驱动的。



### 2、你的第一款 Netty 应用程序

> - 设置开发环境
> - 编写Echo服务器和客户端
> - 构建并测试应用程序



### 3、Netty 的组件和设计

> - Netty的技术和体系结构方面的内容
> - `Channel`、`EventLoop`和`ChannelFuture`
> - `ChannelHandler`和`ChannelPipeline`
> - 引导