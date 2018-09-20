---
title: servlet线程安全
date: 2018-09-20 09:25:54
tags:
- servlet
- java
categories: servlet
---
# servlet 线程安全问题

容器在默认情况下，只会创建一个 **servlet实例**。  
容器收到一个请求，就会 **启动一个线程** 来处理。  

如果有多个请求同时访问某个 servlet，  
就会有多个线程调用同一个 servlet实例，  
就有可能产生线程安全问题（如，这些线程要修改servlet的属性）。   

一个 多线程访问 servlet 示例：  
```java
package web;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ThreadServlet extends HttpServlet {
	private int count = 0;

	@Override
	protected void service(HttpServletRequest arg0, HttpServletResponse arg1) throws ServletException, IOException {
		count++;
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		// 输出线程名+线程个数
		System.out.println(Thread.currentThread().getName() + ":" + count);
	}
}
```

利用 jMeter 测试：  
1、 打开 jMeter，创建一个线程组  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/jMeter1.png)  

2、 配置线程组
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/jMeter2.png)  

3、 为线程组添加 访问协议类型，配置并启动  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/jMeter3.png)  

4、 测试结果  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/jMeter4.png)  

# 解决servlet 线程安全问题  
把可能产生线程不安全的程序 用 **synchronized(this)** 括起来。  
```java
package web;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ThreadServlet extends HttpServlet {
	private int count = 0;

	@Override
	protected void service(HttpServletRequest arg0, HttpServletResponse arg1) throws ServletException, IOException {
		synchronized(this) {
			count++;
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			// 输出线程名+线程个数
			System.out.println(Thread.currentThread().getName() + ":" + count);
		}
	}
}
```
测试结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/jMeter5.png)  
