# SpringMVC 源码解析

## SpringMVC 0xml配置原理

主要使用 javaConfig

## SpringMVC 源码分析

### Spring MVC 找Controller 的流程

1. 扫描整个项目 (Spring 已经做了) 定义一个Map集合
2. 拿到所有加了@Controller注解的类
3. 遍历类里面所有的方法对象
4. 判断方法是否加了 @RequestMapping注解
5. 把@RequestMapping注解的Value 作为map集合的Key给put进去   把method对象作为value放入map集合
6. 根据用户发送的请求   拿到请求中的URI   |---|  URL:http://localhost:8080/test.do  |  URI:test.do
7. 使用请求的uri 作为 map的key 去map里面get  看看是否有返回值

-------------------------------------------------------------------------------------------------------------




## 从发起请求到Controller 中间经过的流程

