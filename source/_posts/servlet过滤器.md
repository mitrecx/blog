---
title: servlet过滤器
date: 2018-09-21 09:25:24
tags:
- servlet
- java
categories: servlet
---
# 过滤器
servlet 规范中 定义的一种特殊的组件，用来拦截容器的调用过程。  

容器收到请求后，通常情况下会调用servlet 的 service 方法来处理请求，  
如果有过滤器，则 **容器先调用过滤器的方法**。  

**注：**  容器一启动，就会创建过滤器。  


## 写一个 过滤器
1、 写一个 java类，实现 **Filter 接口**。  

2、 在 **doFilter方法** 里，编写拦截处理逻辑。  

3、 配置 过滤器 ( web.xml )。  

实例：  
写一个评论页面，在评论框中输入评论，  
如果评论包含“敏感字”就不显示评论，  
如果不包含就显示评论。  

1、 在根目录webContent下，写一个 **comment.jsp** 页面：  
```html
<%@page contentType="text/html; charset=utf-8" pageEncoding="utf-8" %>
<html>
	<head>
	</head>
	<body>
		<form action="comment" method="post">
			评论: <input type="text" name="content"/>
			<input type="submit" value="提交"/>
		</form>
	</body>
</html>
```

2、 写一个 CommentServlet.java， 先不考虑过滤：  
```java
package comment;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CommentServlet extends HttpServlet {
	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		request.setCharacterEncoding("utf-8");
		PrintWriter out = response.getWriter();
		String comment = request.getParameter("content");
		out.println("<h1>你的评论是: " + comment + "</h1>");
		out.close();
	}
}
```
配置一下 **web.xml** ，然后 Tomcat部署，debug ,可以正常访问，下面加入过滤器。  

3、写一个过滤器 **CommentFilterA.java**，屏蔽敏感字"傻" ：  
```java
package web;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class CommentFilterA implements Filter {

	/**
	 * 容器启动后，会创建过滤器对象 只会创建一个
	 */
	public CommentFilterA() {
		System.out.println("FilterA 的构造 方法");
	}

	/**
	 * 容器在创建好 过滤器对象 之后， 接下来会调用对象的init 方法 init 方法只会执行一次 注： 可以在web.xml 中配置
	 * <init-param>提供初始化参数 由 FilterConfig arg 来读取
	 */
	@Override
	public void init(FilterConfig arg0) throws ServletException {
		//可以定义一个属性，来保存 容器(web.xml,init-param)传过来的FilterConfig对象
		//config=arg0
    //在别处 就可以 读初始化参数:
    //String ils = config.getParameter("illegalString");
		System.out.println("FilterA init方法。。");
	}

	/**
	 * 容器调用 doFilter方法来处理请求。
	 *
	 * @param ServletRequest
	 *            arg0 请求对象
	 * @param ServletResponse
	 *            arg1 响应对象
	 * @param FilterChain
	 *            arg2 过滤器链 如果调用 过滤器链 的 doFilter方法 则容器会继续向后调用
	 */
	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {
		System.out.println("FilterA 的 doFilter 方法 开始执行！");
		// 强转，因为要调用的方法是 HttpServletRequest子类的方法
		HttpServletRequest request = (HttpServletRequest) arg0;
		HttpServletResponse response = (HttpServletResponse) arg1;
		// 先执行过滤器，对请求报文 utf-8 编码处理
		request.setCharacterEncoding("utf-8");
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out = response.getWriter();

		String content = request.getParameter("content");
		if (content.indexOf("傻") != -1) {
			// 包含了敏感字
			out.println("<h1>评论包含了敏感字--傻</h1>");
		} else {
			// 没有包含敏感字，继续向后调用
			arg2.doFilter(arg0, arg1);
		}
		System.out.println("FilterA 的 doFilter 方法 执行完毕！");
	}

	@Override
	public void destroy() {
		// TODO Auto-generated method stub
	}
}
```
4、配置 **web.xml** , 添加 过滤器配置：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <display-name>web05-Filter</display-name>
  <filter>
  	<filter-name>filterA</filter-name>
  	<filter-class>web.CommentFilterA</filter-class>
  	<!-- 初始化参数，非必须，通过FilterConfig对象读取 -->
  	<init-param>
  		<param-name>illegalString</param-name>
  		<param-value>猪</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>filterA</filter-name>
  	<url-pattern>/comment</url-pattern>
  </filter-mapping>

  <servlet>
  	<servlet-name>comt</servlet-name>
  	<servlet-class>comment.CommentServlet</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>comt</servlet-name>
  	<url-pattern>/comment</url-pattern>
  </servlet-mapping>
</web-app>
```
5、部署，测试可以实现过滤 "傻" 字。 第一个 **java过滤器** 完成。  

**注：**
当有多个过滤器时，  
容器会依据 **<filter-mapping\>** 在web.xml 中的先后顺序 来调用。在前面的先调用。   

## java过滤器的优点

1、 在不修改源程序的基础上，为程序增加一些过滤条件（过滤器独立于已存在的servlet程序）。   

2、 将多个组件相同的处理逻辑集中写在过滤器里面，方面代码维护。  
比如，request.getSession 做登录验证。  
