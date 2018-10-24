---
title: Spring学习6-过滤器,异常处理,拦截器
date: 2018-10-17 23:59:29
tags: Spring
categories: Spring
---
# 1. 过滤器
过滤器在介绍 servlet 已经 [讲过](https://mitrecx.github.io/2018/09/21/servlet%E8%BF%87%E6%BB%A4%E5%99%A8/)，  
这里在讲是为了抛砖引玉，说明 Spring 提供了一些 **拦截器** ， 在开发中，我们可以直接拿来用。  

用拦截器解决中文乱码问题：  
从页面表单( utf-8 ) 传到 controller( iso-8859 )  的中文是乱码的。  
解决问题很简单，在controller 中加一行代码就能解决。  
```java
request.setCharacterEncoding("UTF-8");
```

考虑到，每个 controller 都会有编码问题，所以写一个 **过滤器** 统一处理比较好。  

写一个类 MyFilter.java  
```java
package cn.mitrecx.filter;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

public class MyFilter implements Filter {
	private String conf;

	@Override
	public void destroy() {
		// TODO Auto-generated method stub

	}

	// 拦截方法
	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
		//前期逻辑
		request.setCharacterEncoding(this.conf);
		//执行chain.doFilter，程序就会继续向后执行
		chain.doFilter(request, response);
		//后期逻辑
	}

	/**
	 * 启动Tomcat服务器时，加载web.xml里的Filter配置
	 */
	@Override
	public void init(FilterConfig arg0) throws ServletException {
		// TODO Auto-generated method stub
		this.conf=arg0.getInitParameter("encoding");
		System.out.println(conf);
	}
}

```
在 web.xml 中配置过滤器  
```xml
<filter>
  <filter-name>myFilter</filter-name>
  <!--  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>-->
  <filter-class>cn.mitrecx.filter.MyFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>myFilter</filter-name>
  <url-pattern>*.do</url-pattern>
</filter-mapping>
```
事实上，上面的这个过滤器Spring 已经写好了，我们直接在web.xml 中配置一下就行了。  
配置方法：  
用  
```xml
<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
```
替换  
```xml
<filter-class>cn.mitrecx.filter.MyFilter</filter-class>
```
即可。  

**CharacterEncodingFilter** 这个类在 spring-web-5.0.0.RELEASE.jar 中，在这个jar 包中有十几个过滤器。  

# 2. 异常处理
异常处理，我们首先能想到的是 try-catch。这里不用try-catch，用Spring 提供的异常处理机制。  
有两种处理方法：  
1. 全局按异常类型处理，遇到异常直接抛出，统一处理。  
2. 局部单独处理。  

就 **第一种处理方法（全局）** 举个例子：  

controller  

```java
package cn.mitrecx.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class ExceptionController {
	@RequestMapping("/exception.do")
	public String execute() throws Exception{
		String ss=null;
		//故意制造一个空指针异常，throws Exception，交个容器处理
		ss.length();
		return "ok";
	}
}
```
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
	<mvc:annotation-driven/>

	<!-- controller -->
	<context:component-scan base-package="cn.mitrecx.controller"></context:component-scan>

	<!-- ViewResolver -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- ViewResolver的前缀后缀属性，用于给controller返回的mav加前后缀 -->
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<!-- 异常处理器 -->
	<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
		<property name="exceptionMappings">
			<props>
				<!-- key是异常类型，value是视图名 -->
				<prop key="java.lang.Exception">error</prop>
			</props>
		</property>
	</bean>
</beans>
```
这样，空指针异常将不会出现一个500页面，而是 WEB-INF/error.jsp 呈现的内容。  

这个流程可以描绘成下面这幅图：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/2018-10-22_223213.png)  

**SimpleMappingExceptionResolver** 是Spring提供的一个简单的异常处理器，我们可以自己写一个，只要实现 **HandlerExceptionResolver接口** 就行。 事实上， SimpleMappingExceptionResolver 也是 实现了 HandlerExceptionResolver接口的。  

这里给一个demo：  
写一个异常处理类MyExceptionHandler.java：  
```java
package cn.mitrecx.exception;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

public class MyExceptionHandler implements HandlerExceptionResolver{

	@Override
	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object arg2, Exception e) {
		ModelAndView mav=new ModelAndView();
		System.out.println("异常信息："+e);
		mav.setViewName("error");
		return mav;
	}
}
```
applicationContext.xml 中配置：  
```xml
<bean class="cn.mitrecx.exception.MyExceptionHandler"></bean>
```

**第二种异常处理方法（局部）**：  
把第一种异常处理方法的 controller 改一下：  
```java
package cn.mitrecx.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class ExceptionController {
	@RequestMapping("/exception.do")
	public String execute() {
		String ss=null;
		//故意制造一个空指针异常，throws Exception，交个容器处理
		ss.length();
		return "ok";
	}

	/**
	 * 这个 异常处理方法只对当前controller起作用，
	 * 方法名自定义，参数固定
	 * @author cx
	 * @param request
	 * @time 下午11:15:33
	 */
	@ExceptionHandler
	public String handleException(HttpServletRequest request, Exception e) {
		return "localError";
	}
}
```
此时，当我访问exception.do 时，  
不会到异常处理页面WEB-INF/error.jsp，  
而会访问异常处理页面WEB-INF/localError.jsp  

# 3. Spring拦截器组件

拦截器组件是 Spring MVC 特有的组件。  
拦截器组件可以在 controller 之前拦截，也可以在 controller 之后拦截。  
还可以在 jsp 解析完成后拦截。  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/spring-interceptor.png)

拦截器使用步骤：  
1. 编写一个拦截器组件，实现 **HandlerInterceptor**  
2. 在applicationContext.xml 中配置拦截器  

实现如下：
编写拦截器组件   
```java
package cn.mitrecx.interceptor;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class LoginInterceptor implements HandlerInterceptor {
	//controll 之前，bool 返回false，就会拦截，拦截器后面的组件就不执行了
	@Override
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
		System.out.println("preHandle:  在controll前拦截。");
		HttpSession session=request.getSession();
		String userName=(String) session.getAttribute("userName");
		//登录过
		if(userName!=null) {
			// 按正常流程走
			return true;
		}else {
			//未登录过，或登录失效
			//重定向到登录页面
			response.sendRedirect("login.do");
			return false;
		}
	}

	// controll 之后
	@Override
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
		System.out.println("postHandle:  controll 之后");
	}

	// 请求出来完毕，输出之前
	@Override
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
		System.out.println("afterCompletion...");
	}
}
```
applicationContext.xml  
```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
	<mvc:interceptor>
		<!-- 拦截哪些请求，/**表示拦截所有请求 /*表示拦截所有不包含子目录的请求 -->
		<mvc:mapping path="/**"/>
		<!-- 放过哪些请求 -->
		<mvc:exclude-mapping path="/login.do"/>
		<mvc:exclude-mapping path="/checkup.do"/>
		<bean class="cn.mitrecx.interceptor.LoginInterceptor"></bean>
	</mvc:interceptor>
</mvc:interceptors>
```
