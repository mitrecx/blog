---
title: Spring学习5-Spring web MVC
date: 2018-10-16 23:38:41
tags: Spring
categories: Spring
---
# 1. Spring Web MVC 简介  
Spring Web MVC 是构建在 Servlet API 上的原始Spring框架。它的正式名称"Spring Web MVC"来自于它的源组件(spring-webmvc)，
但是通常我们简称它为 "Spring MVC" 。  
在Spring Framework 5.0 引入了一个 reactive-stack 的web 框架，其名称为 "Spring WebFlux"，名称同样来自于它的源组件(spring-webflux)，
Spring WebFlux 是与Spring MVC 并行存在的(不是Spring MVC的替代)。  

Spring MVC 依赖于 Spring IoC 。   
Spring MVC 用于 **开发** 具有 MVC 模式的 **web应用**。  

Spring MVC 的 Context Hierarchy(上下文层次体系) 如下，图源来自于Spring官方[参考指南](https://docs.spring.io/spring/docs/5.1.1.RELEASE/spring-framework-reference/web.html#mvc) ：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/mvc-context-hierarchy.png)  

<h1 id='id2'>2. Spring MVC 开发步骤</h1>  

1. 在 **web.xml** 文件中 **<font color='#00B2EE'>配置</font>** **DispatcherServlet**   
2. 在 applicationContext.xml 文件中 **<font color='#00B2EE'>配置</font>** **HandlerMapping**  
3. 编写 **controller** (实现Controller) 在applicationContext.xml中配置注入(或注解注入)  
4. **<font color='#00B2EE'>配置</font>** **ViewResolver** 用于寻找页面展示文件  
5. 编写 页面展示文件(jsp、HTML等)  

注：  
导包需要  
* spring-context-5.0.0.RELEASE.jar
* spring-core-5.0.0.RELEASE.jar
* spring-beans-5.0.0.RELEASE.jar
* commons-logging.jar
* spring-expression-5.0.0.RELEASE.jar
* spring-web-5.0.0.RELEASE-sources.jar
* spring-web-5.0.0.RELEASE.jar
* spring-webmvc-5.0.0.RELEASE.jar
* 扩展，以后开发用
 * standard.jar
 * spring-aop-5.0.0.RELEASE.jar
 * jstl.jar
 * mybatis-3.4.6.jar
 * mysql-connector-java-8.0.12.jar

导包很麻烦？ 先学习下 [Maven]().  

# 3. Spring MVC 入门实例

实例结果:  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/2018-10-17_095628.png)  

实现过程如下：  

1、 在 web.xml 中配置 DispatcherServlet  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <display-name>spring10-16-MVC1</display-name>

  <servlet>
  	<servlet-name>mvc</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>classpath:applicationContext.xml</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
  	<servlet-name>mvc</servlet-name>
  	<url-pattern>*.do</url-pattern>
  </servlet-mapping>

</web-app>
```
2、 编写controller 组件：处理逻辑    
```java
package controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

public class HelloController implements Controller {
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
		ModelAndView mav=new ModelAndView();
    //设置返回页面的名称，此名称会经过ViewResolver加工，最终定位到一个文件
		mav.setViewName("helloSpring");

		//绑定数据，等价于 request.setAttribute
		mav.getModel().put("msg", "由 mav.getModel().put传出");
		return mav;
	}
}
```

3、 配置HandlerMapping 把请求交给对应的 controller 组件    
注入controller ，也可以用注解注入  
配置(注入)ViewResolver 组件，为controller组件定位jsp页面   

applicationContext.xml  
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

	<!-- HandlerMapping -->
	<bean id="handlermapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
		<property name="mappings">
			<props>
        <!-- helloController取自controller的id名 -->
				<prop key="/hello.do">helloController</prop>
			</props>
		</property>
	</bean>

	<!-- controller -->
	<bean id="helloController" class="controller.HelloController">
	</bean>

	<!-- ViewResolver -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- ViewResolver的前缀后缀属性，用于给controller返回的mav加前后缀 -->
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
</beans>
```

4、 展示页面 helloSpring.jsp  
```html
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8" %>
<html>
	<head>
	</head>

	<body style="font-size: 30px;">
		hello Spring web MVC.  <br/>
		<br/> msg: ${msg }
		<br/> requestScope.msg: ${requestScope.msg }
	</body>
</html>
```

# 4. 使用注解改进程序  
Spring MVC 使用注解开发步骤和[前面步骤](#id2)一样，只是：  
把applicationContext.xml 中的 **handlerMapping** 这个 bean 用 **mvc:annotation-driven** 代替,  
把 **配置controller** 用 **扫描context:component-scan** 代替，  
controller类里加上注解。  

使用注解的好处是方便灵活。   

程序实现：  
web.xml 不变。  

applicationContext.xml  
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

	<!-- HandlerMapping : 用mvc:annotation-driven包含了多个bean，其中有handlerMapping这个bean -->
	<mvc:annotation-driven/>

	<!-- 扫描controller -->
	<context:component-scan base-package="controller"></context:component-scan>

	<!-- ViewResolver -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- ViewResolver的前缀后缀属性，用于给controller返回的mav加前后缀 -->
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
</beans>
```
controller组件 helloSpring.java  
```java
package controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
@Controller
public class HelloController {
	@RequestMapping("/hello.do") //直接把请求和处理方法关联
	public ModelAndView execute() {
		ModelAndView mav = new ModelAndView();
		mav.getModel().put("msg", "注解版");
    //返回视图名称
		mav.setViewName("helloSpring");
		return mav;
	}

  //可以注解多个RequestMapping
}
```  
页面 helloSpring.jsp  
```html
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8" %>
<html>
	<head>
	</head>

	<body style="font-size: 30px;">
		hello Spring web MVC.  <br/>
		<br/> msg: ${msg }
	</body>
</html>
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/spring-mvc-2.png)
