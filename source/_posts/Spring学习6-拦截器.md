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
