# Mybatis源码分析

## mybaist 的流程分析

首先 `mybatis` 的源码分为两种情况:

1. 单独的 `mybaits`
2. 和 `spring` 整合的 `mybatis` 的源码

这两种情况下的源码分析会有点不同，比如如果是分析 `mybatis + spring` 的模式那么 `mybaits` 的开始就是从 `spring` 初始化的时候开始。本文主要针对这种情况来分析 `mybatis` 的流程。

首先考虑 `spring + mybatis` 的时候他们的关联点有哪些?

1. `@MapperScan`
2. `@Bean SqlSessionFactoryBean`

如果你精通 spring 或者你去看一下 `@MapperScan` 的源码就会就会知道他所利用就是 spring 当中的 `Import` 和
`ImportBeanDefinitionRegistrar` 技术来对 spring 进行了扩展，在对比一下 `@BeanSqlSessionFactoryBean` 就会知道 spring 会首先去执行 `ImportBeanDefinitionRegistrar` 当中的 `registerBeanDefinitions` 方法，关于这一块的代码在上次讲 spring 源码的时候以及分析过，故而直接过了。

## mybatis 的一级缓存的各种问题

### spring 当中为什么失效

因为 mybatis 和 spring 的集成包当中扩展了一个类 `SqlSessionTemplate` ，这个类在 spring 容器启动的时候被注入给了 mapper 这个类替代了原来的 `DefulatsqlSession`，`SqlSessionTemplate` 当中的所有查询方法不是直接查询，而是经过一个代理对象，代理对象增强了查询方法，主要是关闭了session

### mybaits的级缓存技术的底层原理