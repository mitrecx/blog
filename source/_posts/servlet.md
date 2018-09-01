---
title: servlet
date: 2018-09-01 21:45:59
tags:
- servlet
- java
categories: servlet
---
# 1. 什么是servlet ？  
sun公司制定的一种用来**扩展web服务器功能** 的 **组件规范**。  
（1）扩展web服务器功能  
&nbsp;&nbsp;&nbsp;web服务器只能处理静态资源的请求（即事先需要把HTML文件准备好），可以使用servlet来扩展，即web服务器可以通过调用servlet来处理动态资源的请求，比如访问数据库。    
（2）组件规范  
&nbsp;&nbsp;&nbsp;**组件**：符合一定规范，实现部分功能，并且需要部署到相应的容器内才可以运行的软件模块。  
&nbsp;&nbsp;&nbsp;**servlet**是一个容器，需要部署到相应的servlet容器里才能运行。  
&nbsp;&nbsp;&nbsp;**容器**：提供组件运行环境的程序。  

# 2. 如何写一个servlet  
不利用开发工具，纯手写一个servlet：  
1. 写一个java类，继承Servlet接口或者**继承HttpServlet抽象类**。  
2. 编译
3. 打包（变成一个组件）  
创建具有如下结构的文件夹：  
APPname/（应用名）  
 &nbsp;&nbsp; WEB-INFO/  
 &nbsp;&nbsp;&nbsp;&nbsp; classes/  
 &nbsp;&nbsp;&nbsp;&nbsp; lib/  (可选，放jar文件)  
 &nbsp;&nbsp;&nbsp;&nbsp; **web.xml** (**部署描述文件**)
4. 部署  
将第3步创建好的整个文件夹拷贝到servlet容器的相应的位置。  
<font color=red>注</font>： 可以使用 **jar** 命令将第3步创建好的整个文件夹压缩成 **".war"** 的文件，然后拷贝。  
5. 启动容器，访问servlet  
http://ip:port/APPname/url-pattern  
<font color=red>注</font>： **url-pattern** 在web.xml 文件中定义  


# 3. Tomcat是如何运行的：  
在浏览器地址栏输入一个地址  
step1.&nbsp;  browser 根据ip, port 建立连接。  
step2.&nbsp; browser 将相关数据 (请求参数) 打包，然后发送。  
step3.&nbsp; Tomcat 解析请求数据包，并且将解析到的数据封装到 request。 对象，同时创建一个response对象。  
step4.&nbsp; Tomcat容器创建 servlet 对象，然后调用该对象的 service() 方法。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <font color=red>注</font>：Tomcat 容器会将 request 和 response 传递，通过 request 获得请求参数，将处理结果写到 response 。  
step5.&nbsp; Tomcat 容器读取 response 中的处理结果，并将处理结果打包发送给 browser。  
step6.&nbsp; 浏览器解析相应数据包，生成相应的页面。  

# 4. HTTP 状态码：  
404 :    
<font color=red>HTTP Status 404 – Not Found</font> 没有资源  
描述：The requested resource is not available.   
或者是，The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.  
<font color=purple>可能原因：</font>  
1. 请求地址写错  
2. 没有部署该应用
3. map.xml 中，<servlet-name> 不一致

500 :  
<font color=red>HTTP Status 500 – Internal Server Error</font> 服务器程序运行错误   
cannot be cast to javax.servlet.Servlet  
<font color=purple>可能原因：</font>  
1. 我们自己写的 servlet 没有继承 HTTPServlet. (console: cannot be cast to javax.servlet.Servlet)  
2. map.xml 中，<servlet-class> 写错了，包中没有这个 class。  
3. 代码写的不严谨，比如 对请求参数没有做检查就去转换。  


# 5. 请求参数  
没有请求参数：  
http://localhost:8080/TestTomcat2/wowowo  
有请求参数：  
http://localhost:8080/TestTomcat2/wowowo<font color=red>?num=10</font>  
**问号** 表示 后面是一些请求参数。num是请求参数名。   
http://localhost:8080/TestTomcat/bmi<font color=red>?weight=60&height=1.9</font>   
**&** 连接多个请求参数。  




# 6. 字符集编码问题

servlet 输出中文为什么会有乱码？  
out.println 方法在默认情况下，会使用"iso-8859-1"来编码  

解决：  
response.setContentType("text/html;charset=utf-8")  
<font color=red>注</font>：先设置编码，后获取输出流 response.getWriter()  

表单包含中文参数值 出现乱码：  
**原因**：  表单提交时，browser会对表单中的值进行编码（用页面的编码方式），服务端解码时，默认用 iso-8859-1 来解码  
**解决**：
1. 在HTML 的 <head\> 中加个<meta> 标签，指定表单提交时的编码方式     
<meta http-equiv="content-type" content="text/html; charset=utf-8"\>   
2. 服务端使用相同的 **字符集** 解码  
```java
request.setCharacterEncoding("utf-8");  
```

# 7. 转发  
1、 转发：  
一个 **web 组件 (servlet/jsp)** 将为完成的处理交个另外一个 **web 组件** 继续做。  
比如，一个 servlet 将处理结果 转发 给一个 jsp 来展现。  

2、 如何转发：  
1. 绑定数据到 request 对象上   

```java
//name 绑定名， obj 绑定值
//obj 不存在的话 就是null
request.setAttribute(String name, Object Obj);

//依据绑定名获得绑定值
Object request.getAttribute(String name);

```

2. 获得转发器  

```java
//uri 是转发的地址
RequestDispatcher rd= request.getRequestDispatcher(String uri);
```

3. 转发  

```java
rd.forward(request, response);  
```

注：  
* 转发之后 ， 浏览器的地址不变。即，浏览器访问的地址，不会随着转发而改变，转发是服务器内部的事情，浏览器并不知情。  
* 转发地址有限制，必须是同一个应用。  
