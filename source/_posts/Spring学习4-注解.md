---
title: Spring学习4-注解
date: 2018-10-15 18:50:02
tags: Spring
categories: Spring
---

**JDK 5.0** ，是JDK (Java Development Kit) 具有里程碑的一个版本。2004年发布的。  
SE(JavaSE): standard edition，从JDK 5.0开始，改名为Java SE。  
EE(JavaEE): enterprise edition，从JDK 5.0开始，改名为Java EE。  
JDK 5.0 引入许多新特性，包括泛型(Generic)、可变参数、for-each、**注解** 等。  

**注解** : 在 类的定义，方法的定义，成员变量的定义 前使用，格式为 **@注解标记名** 。   

# 1. 组件扫描 (component-scan)  
可以根据包的路径，把包下所有的组件扫描，如果发现组件类定义前有以下标记，将会把组件扫描的Spring IoC 容器，这样就不用同一个包下的组件就不用一个一个的在主配置文件中配置bean了。     
* @Controller  
* @Service  
* @Repository  
* @Component
* @Named---需要引入第三方标准包  

以上的注解可以混用，但是一般我们按类使用：     
* @Controller 用于扫描控制层组件  
* @Service 扫描业务层组件  
* @Repository 扫描数据访问层组件  
* @Component 扫描其他组件  

一个组件扫描的例子：  
在applicationContext.xml中，开启组件扫描  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
		http://www.springframework.org/schema/util
		http://www.springframework.org/schema/util/spring-util-4.3.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

	<!-- 扫描包controller下的组件 -->
	<context:component-scan base-package="controller"/>

</beans>
```
包controller下的组件LoginController.java ：  
```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.context.annotation.Scope;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Controller //注解: 扫描到Spring容器
@Scope("prototype") //等价于<bean scope="prototype"> , 默认singleton
public class LoginController {

	public void execute() {
		System.out.println("处理逻辑");
	}

  //了解一下
  @PostConstruct //等价于 init-method
  public void init() {
    System.out.println("初始化逻辑");
  }
  @PreDestroy //等价于 destroy-method
  public void destroy() {
    System.out.println("销毁");
  }
}
```
注：  
扫描到Spring IoC容器中的组件 id 默认类名并且首字母小写。  
如果想要改变 id 名， 可以在注解后加指定的id名：  
**@Controller(loginComponent)**    

在举个例子，  
利用注解注入属性：  
```java
package other;
import org.springframework.stereotype.Component;
import javax.annotation.Resource;

@Component
public class Student {
  @Resource //注入 id 为 phone 的对象
  private Phone phone;

  //@Resource(name="指定id") 可以注入指定id的对象

  //@Autowired 按类型注入
}
```
