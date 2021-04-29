# Mybatis 日志

## 1、mybatis + log4j 可以打印日志

Mybatis 控制日志 是 LogFactory 类

其中有静态代码块

```java
static {
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useSlf4jLogging();
    }
  });
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useCommonsLogging();
    }
  });
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useLog4J2Logging();
    }
  });
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useLog4JLogging();
    }
  });
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useJdkLogging();
    }
  });
  tryImplementation(new Runnable() {
    @Override
    public void run() {
      useNoLogging();
    }
  });
}
```

可知 其 默认是使用 SLF4J 的

因此 当 mybatis 中引用了 log4j 时，mybatis 可以打印日志



## 2、mybatis + spring + log4j  不可以打印日志

准确来说是 Mybatis + Spring5 + log4j 不可以打印日志

这里就要说到 **spring5 日志的新特性**了

Spring5 之前 默认使用 JCL 打印日志，JCL 默认是使用 Log4j 打印日志的，当项目中没有 Log4j 时会使用 JUL 打印日志，所以当 项目中有依赖 log4j 时，JCL 会使用 Log4j 打印日志。因此 Mybatis + Spring5之前 + log4j 可以打印日志

Spring5 使用的是 SpringJCL 打印日志，其默认是使用 JUL 打印日志的。

**但是为什么说  Mybatis + Spring5 + log4j 不可以打印日志呢？**

因为 JUL 的日志级别

JUL 默认只打印 INFO 以上的日志，而 mybatis 是使用 INFO 以下的日志级别，因此无日志输出

**那 mybatis 为什么会 使用 spring 的日志输出呢？**

这里就要说到上面的源码 LogFactory 的静态代码块

使用日志顺序如下

SLF4J 》JCL 》Log4j2 》Log4J 》JUL

因此会使用到 spring 中默认携带的 JCL 进行日志的打印

