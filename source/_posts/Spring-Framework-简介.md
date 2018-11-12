---
title: Spring学习1-Spring Framework 简介
date: 2018-10-12 01:25:37
tags: Spring
categories: Spring
---
# 1. 为什么选择Spring 框架
如果没有Spring 框架，我们按照 MVC模式 开发一个 web应用，需要配置web.xml，写Servlet来处理请求，写实体类和DAO类，写jsp展示页面等。  
这样做思路和结构都很清晰，没有什么不好。  
那为什么我们还要用Spring 框架？  
原因：  
1. **松耦合**--IoC, AOP。    
2. **面向切面编程(AOP)**--组件复用，将系统服务(安全、事务、日志等)从业务逻辑中分离出来。  
3. **简化web开发**--缩短开发时间。  
可见，Spring 框架的设计哲学是优秀的，于是许多开发者选择了Spring web MVC。  

# 2. 了解一下-术语Spring
不同的背景下，Spring的含义是不同的。  
通常我们说的 **Spring** 指的是 **[Spring项目家族](https://spring.io/projects)** ，包含Spring boot，Spring Framework, Spring Cloud等等。  
这个Spring系列的博客，指的是Spring Framework。  
没有特殊说明的话，后面Spring 默认指的是Spring Framework。  

Spring Framework被分离成多个模块，我们开发的web应用可以选择自己需要的模块。  
这些模块的核心是 Core Container，**包含** 配置模块和依赖注入机制(dependency injection mechanism)。  
此外，Spring Framework 支持不同的应用结构(application architecture)：  
* messaging(消息传递)  
* transactional data and persistence(事务数据和持久化)  
* web  

**还包含** 基于 **<font color='#00B2EE'>Spring MVC</font>** 的web框架 和 **<font color='#00B2EE'>Spring WebFlux</font>** 响应式的web框架。  

# 3. Spring 的版本历史

|Version|	Date|	Notes|
|---|---|---|
|0.9|	2002|[Rod Johnson]()开发了第一版|
|1.0|	2003|里程碑|
|2.0|	2006||
|3.0|	2009||
|4.0|	2013|JavaSE 8，Groovy 2，JavaEE 7|
|5.0|	2017|WebFlux，响应式编程|  

# 参考资料  
1、[Spring Framework Overview](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/overview.html#overview)  
2、[ Spring Projects](https://spring.io/projects)  
3、[SSH框架——Spring + Struts + Hibernate简介-2016](https://www.cnblogs.com/wt695742319/p/5500392.html)  
4、[SSM框架——详细整合教程(Spring + SpringMVC + MyBatis)](https://www.cnblogs.com/zyw-205520/p/4771253.html)
