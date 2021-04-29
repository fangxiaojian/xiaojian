# Spring

## Spring容器的启动流程：

（1）初始化Spring容器，注册内置的BeanPostProcessor的BeanDefinition到容器中：

① 实例化BeanFactory【DefaultListableBeanFactory】工厂，用于生成Bean对象
② 实例化BeanDefinitionReader注解配置读取器，用于对特定注解（如@Service、@Repository）的类进行读取转化成  BeanDefinition 对象，（BeanDefinition 是 Spring 中极其重要的一个概念，它存储了 bean 对象的所有特征信息，如是否单例，是否懒加载，factoryBeanName 等）
③ 实例化ClassPathBeanDefinitionScanner路径扫描器，用于对指定的包目录进行扫描查找 bean 对象
（2）将配置类的BeanDefinition注册到容器中：

（3）调用refresh()方法刷新容器：

① prepareRefresh()刷新前的预处理：
② obtainFreshBeanFactory()：获取在容器初始化时创建的BeanFactory：
③ prepareBeanFactory(beanFactory)：BeanFactory的预处理工作，向容器中添加一些组件：
④ postProcessBeanFactory(beanFactory)：子类重写该方法，可以实现在BeanFactory创建并预处理完成以后做进一步的设置
⑤ invokeBeanFactoryPostProcessors(beanFactory)：在BeanFactory标准初始化之后执行BeanFactoryPostProcessor的方法，即BeanFactory的后置处理器：
⑥ registerBeanPostProcessors(beanFactory)：向容器中注册Bean的后置处理器BeanPostProcessor，它的主要作用是干预Spring初始化bean的流程，从而完成代理、自动注入、循环依赖等功能
⑦ initMessageSource()：初始化MessageSource组件，主要用于做国际化功能，消息绑定与消息解析：
⑧ initApplicationEventMulticaster()：初始化事件派发器，在注册监听器时会用到：
⑨ onRefresh()：留给子容器、子类重写这个方法，在容器刷新的时候可以自定义逻辑
⑩ registerListeners()：注册监听器：将容器中所有的ApplicationListener注册到事件派发器中，并派发之前步骤产生的事件：
⑪  finishBeanFactoryInitialization(beanFactory)：初始化所有剩下的单实例bean，核心方法是preInstantiateSingletons()，会调用getBean()方法创建对象；
⑫ finishRefresh()：发布BeanFactory容器刷新完成事件：