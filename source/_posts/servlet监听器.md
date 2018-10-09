---
title: servlet监听器
date: 2018-09-22 09:25:36
tags:
- servlet
- java
categories: servlet
---
# 监听器

监听器 是 servlet规范 中定义的一种特殊的组件，**用来监听容器产生的事件**。  

容器产生的事件主要有两大类：  
1、 生命周期相关的事件  
容器创建或销毁了 request，session，**servlet上下文** 时产生的事件。  

2、 绑定数据相关的事件  
调用了 request，session，**servlet上下文** 的 setAttribute，removeAttribute 时产生的事件。  

## 1. servlet 上下文

**什么是 servlet上下文 ？**    
  容器启动之后，**会为每一个web应用 创建唯一的一个符合ServletContext接口的对象**。  

### 1.1 <font color=orange>特点</font>  
1、 唯一性： 一个web应用 对应唯一一个上下文。  
2、 持久性： 只要容器没有关闭，并且应用没有被删除，则上下文会一直存在。  

获得上下文的 **4种途径**：    
**<font color=green>GenericServlet，或 ServletConfig，或 FilterConfig，或 HttpSession</font>**   
提供的 **getServletContext** 方法。  

### 1.2 <font color=orange>上下文的作用</font>：  
将数据绑定到上下文，应用可以随时访问。  
1、绑定数据，  
setAttribute，getAttribute，removeAttribute  
注：  
request.setAttribute, session.setAttribute, ServletContext.setAttribute, **在绑定数据时，应该优先使用生命周期短的对象绑定**，这样可以节约内存空间( request < session < context )。  

例如：  
在 ServletA对象中绑定 一条数据，在ServletRead对象中读取并显示。  
ServletA.java  
```java
package web;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ContextA extends HttpServlet{
	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out=response.getWriter();
		// 获得上下文context，注：HTTPServlet 是 GenericServlet的子类
		ServletContext ctx= getServletContext();

		// 将一条数据绑定到 context
		ctx.setAttribute("userList", "陈兴，mitre");
		out.close();
	}
}

```
ContextRead.java  
```java
package web;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ContextRead extends HttpServlet {
	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out=response.getWriter();
		//获得上下文context
		ServletContext ctx=getServletContext();

		// 读取context绑定的数据
		String ul=(String)ctx.getAttribute("userList");
		out.println(ul);
		out.close();
	}
}

```
web.xml  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <display-name>web06-context</display-name>

  <servlet>
  	<servlet-name>write</servlet-name>
  	<servlet-class>web.ContextA</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>write</servlet-name>
  	<url-pattern>/set</url-pattern>
  </servlet-mapping>

  <servlet>
  	<servlet-name>read</servlet-name>
  	<servlet-class>web.ContextRead</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>read</servlet-name>
  	<url-pattern>/get</url-pattern>
  </servlet-mapping>

</web-app>
```

2、 访问全局的初始化参数。  
web.xml配置：<context-param\>  
ServletContext对象读取: String getInitParameter(String paramterName)  

例如：  
接着上一个例子  
在 web.xml 配置：  
```xml
<!-- 全局的初始化参数 -->
<context-param>
  <param-name>company</param-name>
  <param-value>宇信</param-value>
</context-param>
```
在 ServletRead.java 中读取初始化参数(添加两行代码):  
```java
package web;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ContextRead extends HttpServlet {
	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out=response.getWriter();
		//获得上下文context
		ServletContext ctx=getServletContext();

		// 读取context绑定的数据
		String ul=(String)ctx.getAttribute("userList");
		out.println(ul);

		// 读取 全局的初始化参数
		String companyName=ctx.getInitParameter("company");
		out.println("<br/>"+companyName);
		out.close();
	}
}

```
执行结果:  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/servlet-context-01.png)  

## 2. 监听器

写一个监听器：  
1、 写一个 java类，实现相应的监听器接口。  
注：   
要依据监听的事件类型来选择合适的接口。  
比如，要监听 **session** 的创建和销毁，需要实现 **HTTPSessionListener接口**。  

2、 实现接口方法，完成监听处理逻辑。  

3、 配置 web.xml  

实例：  
使用监听器统计在线人数。  

配置web.xml  
```xml
<!--首页-->
<welcome-file-list>
  <welcome-file>index.jsp</welcome-file>
</welcome-file-list>

<!--其他servlet配置-->

<!--退出-->
<servlet>
  <servlet-name>logout</servlet-name>
  <servlet-class>web.LogOutServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>logout</servlet-name>
  <url-pattern>/logout</url-pattern>
</servlet-mapping>
```
退出界面/首页 index.jsp  
```html
<%@ page language="java" contentType="text/html; charset=utf-8"
    pageEncoding="utf-8"%>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
	<title>index</title>
</head>
<body>
	<h1>这是首页</h1>
	当前系统在线人数：
	<%= application.getAttribute("count") %>
	<a href="logout" >退出</a>
</body>
</html>
```

退出登录(销毁HttpSession): LogOutServlet.java   
```java
package web;
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

public class LogOutServlet extends HttpServlet{
	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		HttpSession hSession=request.getSession();
		hSession.invalidate();
	}
}
```
