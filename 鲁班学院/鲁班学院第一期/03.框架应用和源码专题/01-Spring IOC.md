# Spring IOC

## what is IOC

> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）

### Dependency Injection

- 依赖注入
  - 关于什么是依赖
    - 依赖指的是两个实例之间的关系。其中一个实例是独立的，另一个实例是非独立的（依赖的），它依靠另外一个实例。比如计算机对象，它包含主机对象和显示器对象。如果没有主机对象或者显示器对象，则计算机对象就是不完整的，不能正常使用，我们就说计算机对象依赖于主机对象和显示器对象。 
  - 关于注入和查找以及拖拽

## 为什么要使用spring IOC

spring体系结构----IOC的位置  自己看官网

> 在日常程序开发过程当中，我们推荐面向抽象编程，面向抽象编程会产生类的依赖，当然如果你够强大可以自己写一个管理的容器，但是既然spring以及实现了，并且spring如此优秀，我们仅仅需要学习spring框架便可。

> 当我们有了一个管理对象的容器之后，类的产生过程也交给了容器，至于我们自己的app则可以不需要去关系这些对象的产生了。

## spring实现IOC的思路和方法

spring实现IOC的思路是提供一些配置信息用来描述类之间的依赖关系，然后由容器去解析这些配置信息，继而维护好对象之间的依赖关系，前提是对象之间的依赖关系必须在类中定义好，比如A.class中有一个B.class的属性，那么我们可以理解为A依赖了B。既然我们在类中已经定义了他们之间的依赖关系那么为什么还需要在配置文件中去描述和定义呢？

spring实现IOC的思路大致可以拆分成3点

1. 应用程序中提供类，提供依赖关系（属性或者构造方法）
2. 把需要交给容器管理的对象通过配置信息告诉容器（xml、annotation，javaconfig）
3. 把各个类之间的依赖关系通过配置信息告诉容器



配置这些信息的方法有三种分别是xml，annotation和javaconfig

维护的过程称为自动注入，自动注入的方法有两种构造方法和setter

自动注入的值可以是对象，数组，map，list和常量比如字符串整形等

## spring编程的风格

`````java
schemal-based-------xml

annotation-based-----annotation

java-based----java Configuration
`````

## 注入的两种方法

spring注入详细配置（字符串、数组等）[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-properties-detailed)

### Constructor-based Dependency Injection

构造方法注入[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-constructor-injection)

![img](image\01-1.png!original)

### Setter-based Dependency Injection

setter[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-setter-injection)

![img](image/01-2.png!original)

## 自动装配

上面说过，IOC的注入有两个地方需要提供依赖关系，一是类的定义中，二是在spring的配置中需要去描述。自动装配则把第二个取消了，即我们仅仅需要在类中提供依赖，继而把对象交给容器管理即可完成注入。

在实际开发中，描述类之间的依赖关系通常是大篇幅的，如果使用自动装配则省去了很多配置，并且如果对象的依赖发生更新我们可以不需要去更新配置，但是也带来了一定的缺点

自动装配的优点[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)

缺点[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-autowired-exceptions)

作为我来讲，我觉得以上缺点都不是缺点

### 自动装配的方法

````java
no   // 不使用 == dafault
byType
byName
constructor
````

#### 全局指定

在xml文件的beans中 指定 default-autowire=“no”>

#### 单独指定

只指定需要的bean

在bean中配置 autowire=“no” 属性值

单独指定权限高于全局指定

#### 官方文档

自动装配的方式[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-autowire)

#### byType

spring会根据你使用的类名去bean中找他的实现类，找到就自动装配

缺点：

若有两个实现类实现同一个接口，并且这两个实现类都交由spring进行管理，自动装配时就会报错

#### byName

spring会根据被需要依赖的类如service 中set的注入方法名 去匹配bean标签中的name属性值或者id

![img](image\01-3.png!original)



```java
@Resource
private IndexDao indexDaoImpl1;
```

默认是根据属性名去添加依赖的，即会装配IndexDaoImpl1 这个实现类。

先根据byName查找，找不到再根据byType。

> @Resource(type=IndexDaoImpl1.class)

指定对象



```java
@Autowired
private IndexDao indexDaoImpl1;
```

与byType相似。先根据byType查找，找不到时再根据byName。



## spring懒加载

官网已经解释的非常清楚了：

[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lazy-init)

值得提醒的是，如果你想为所有的对都实现懒加载可以使用官网的配置

![img](image\01-4.png!original)

## springbean的作用域

[文档参考](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-scopes)

![img](image\01-5.png!original)

xml定义方式

<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>

annotation的定义方式

### Singleton Beans with Prototype-bean Dependencies

@Scope("singleton") // spring默认就是单例的

意思是在Singleton 当中引用了一个Prototype的bean的时候引发的问题

官网引导我们[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-method-injection)

service 是单例的

dao       是多例的

在service中依赖的dao就会出现单例的情况

解决方法一：

service 实现 ApplicationContextAware 接口，通过内部 ApplicationContext 属性 getBean 方法获取依赖

```java
@Service("service")
public class IndexService  implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    private IndexDao dao0;

    public void service(){
        dao0 = (IndexDaoImpl1) applicationContext.getBean("dao1");
        dao0.test();
        System.out.println("service = " + this.hashCode());
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

缺点：耦合度高



解决方法二：

使用@Lookup 注解

```java
@Service("service1")
public abstract class IndexService1{

    @Lookup("dao1")
    public abstract IndexDao getDao();

    private IndexDao dao0;

    public void service(){
        dao0 = getDao();

        dao0.test();
        System.out.println("service = " + this.hashCode());
    }

}
```



## spring生命周期和回调

[参考文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle)

1、Methods annotated with @PostConstruct

2、afterPropertiesSet() as defined by the InitializingBean callback interface

3、A custom configured init() method