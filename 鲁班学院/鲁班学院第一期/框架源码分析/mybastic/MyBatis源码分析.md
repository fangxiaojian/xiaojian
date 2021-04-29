# **Mybatis** 源码分析

## mybatis 的流程分析

首先 `mybatis` 的源码分为两种情况:

1. 单独的 `mybatis`
2. 和 `spring` 整合的 `mybatis` 的源码

这两种情况下的源码分析会有点不同, 比如如果是分析 `mybatis` + `spring` 的模式那么 `mybatis` 的开始就是从 `spring` 初始化的时候开始. 

本文主要针对这种情况来分析 `mybatis` 的流程.

首先考虑 `spring` + `mybatis` 的时候他们的关联点有哪些?

1. `@MapperScan`
2. `@BeanSqlSessionFactoryBean`

如果你精通 `spring` 或者你去看一下 `@MapperScan` 的源码就会知道他所利用的就是 `spring` 当中的 `Import` 和 `ImportBeanDefinitionRegistrar` 技术对 `spring` 进行了扩展, 在对比一下 `@BeanSqlSessionFactoryBean` 就会知道 `spring` 会首先去执行 `ImportBeanDefinitionRegistrar` 当中的 `registerBeanDefinitions` 方法, 关于这一块的代码在上次讲 `spring` 源码的时候就分析过, 故而直接过了.



## mybatis 的一级缓存的各种问题

### spring 当中为什么失效



### mybatis 的一级缓存技术的底层原理





## $ 与 #

### ${}----->动态的sql语句

只要拥有$ 就是动态sql

select * from user where id = ${id}

select * from user where id = ${id} and username = #{username}

解析时sql语句不变

select * from user where id = ${id}

select * from user where id = ${id} and username = #{username}

执行时才将 ${} 赋值

select * from user where id = 1

select * from user where id = 1 and username = ?



### #{}----->静态的sql语句

select * from user where id = #{id}

解析sql语句时就生成占位符?

select * from user where id = ?

真正执行sql语句时才对占位符? 进行赋值

select * from user where id = 1

