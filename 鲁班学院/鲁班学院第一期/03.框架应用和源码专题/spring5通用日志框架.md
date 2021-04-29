# spring5通用日志框架

https://shimo.im/docs/kKCgwqgQqHYTeQ8J/read

- 各种日志技术的关系和作用

- 通过源码来分析spring的日志技术

- commons-logging源码分析

- 通过源码分析mybaits的日志技术

- 架构系统时候如何选择、优化日志技术





## 主流的log技术名词

1. log4j

自动检测

<!--<dependency>-->

​    <!--<groupId>log4j</groupId>-->

​    <!--<artifactId>log4j</artifactId>-->

​    <!--<version>1.2.12</version>-->

<!--</dependency>-->

可以不需要依赖第三方的技术

直接记录日志

1. jcl

jakartaCommonsLoggingImpl

自动检测

<dependency>

​    <groupId>commons-logging</groupId>

​    <artifactId>commons-logging</artifactId>

​    <version>1.1.1</version>

</dependency>

jcl他不直接记录日志,他是通过第三方记录日志(jul)



如果使用jcl来记录日志,在没有log4j的依赖情况下,是用jul

如果有了log4j则使用log4j

自动检测

jcl=Jakarta commons-logging ,是apache公司开发的一个抽象日志通用框架,本身不实现日志记录,但是提供了记录日志的抽象方法即接口(info,debug,error.......),底层通过一个数组存放具体的日志框架的类名,然后循环数组依次去匹配这些类名是否在app中被依赖了,如果找到被依赖的则直接使用,所以他有先后顺序。

![img](https://images-cdn.shimo.im/yFBaGU5euKMuCqtu/jcl1.png!thumbnail)



自动检测

上图为jcl中存放日志技术类名的数组，默认有四个，后面两个可以忽略。

![img](https://images-cdn.shimo.im/WUYYvMBBkx4eL28s/jcl0.png!thumbnail)

自动检测

上图81行就是通过一个类名去load一个class，如果load成功则直接new出来并且返回使用。

如果没有load到class这循环第二个，直到找到为止。

![img](https://images-cdn.shimo.im/8MzEr8n4FycGtto5/jcl2.png!thumbnail)

自动检测

可以看到这里的循环条件必须满足result不为空，

也就是如果没有找到具体的日志依赖则继续循环，如果找到则条件不成立，不进行循环了。

总结：顺序log4j>jul



1. jul

java自带的一个日志记录的技术,直接使用

1. log4j2

1. **slf4j**

**slf4j他也不记录日志,通过绑定器绑定一个具体的日志记录来完成日志记录**

自动检测

log4j---<artifactId>slf4j-log4j12</artifactId>

1. logback

1. simple-log

## 各种日志技术的关系和作用

![img](https://images-cdn.shimo.im/HCYt068pb1YY5GZZ/log.png!thumbnail)

## spring日志技术分析

### spring5.*日志技术实现

spring5使用的spring的jcl(spring改了jcl的代码)来记录日志的,但是jcl不能直接记录日志,采用循环优先的原则

###  spring4.*日志技术实现

spring4当中依赖的是原生的jcl

## mybatis的日志技术实现

### 初始化

**org.apache.ibatis.logging.LogFactory**



自动检测

tryImplementation(new Runnable() {

  @Override

  public void run() {

​    useSlf4jLogging();

  }

});

关键代码     **if (logConstructor == null)****,**没有找到实现则继续找

自动检测

private static void tryImplementation(Runnable runnable) {

  if (logConstructor == null) {

​    try {

​      runnable.run();

​    } catch (Throwable t) {

​      // ignore

​    }

  }

}

![img](https://images-cdn.shimo.im/pNtdoqBmmD0mUXLE/my0.png!thumbnail)![img](https://images-cdn.shimo.im/5bPdBJeaxZ0vZT3J/my1.png!thumbnail)



### 具体实现类

mybatis提供很多日志的实现类,用来记录日志,取决于初始化的时候load到的class

![img](https://images-cdn.shimo.im/26jOIrgomZIDKJ5f/my2.png!thumbnail)

自动检测

上图红色箭头可以看到JakartaCommonsLoggingImpl

中引用了jcl 的类,如果在初始化的时候load到类为JakartaCommonsLoggingImpl

那么则使用jcl 去实现日志记录,但是也是顺序的,顺序参考源码



### 自己模拟实现mybaits的日志实现

自动检测

mybatis的官网关于日志的介绍

定义org.apache.ibatis.session.Configuration

参考org.apache.ibatis.logging.commons.JakartaCommonsLoggingImpl

分析为什么jcl不记录日志,修改代码



## 架构系统如何考虑日志

old:jcl+log4j

new:slf4j+jul