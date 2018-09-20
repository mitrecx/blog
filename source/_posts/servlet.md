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




# 4. 请求资源路径 和 请求参数  
## 4.1 web.xml 配置请求资源路径
容器如何处理请求资源路径：  
在浏览器地址栏输入 http://ip:port/web_App_name/hello.html  
容器根据 **应用名** 找到 应用所在的目录  
容器默认调用一个servlet，去 web.xml 中查找有没有一个和"/hello.html" 匹配的servlet。具体调用按下列顺序：  
1. 如果 web.xml 中有，就调用此servlet。   
2. 如果 web.xml 中没有，就调用 WebContent(应用主目录) 目录 下的 对应的"/hello.html" 文件。  
3. 如果 Webcontent 目录下也没有资源文件， 将会报 404 。  

web.xml 中配置 <url-pattern\> </url-pattern\> 有三种匹配方式：  
1、 精确匹配  
<url-pattern\> /hello.html </url-pattern\>  

2、 通配符匹配  
<url-pattern\> /\* </url-pattern\>  
使用 " **\*** " 匹配 0 或 多个 字符  

3、 **后缀匹配**  
<url-pattern\> **\*.do** </url-pattern\>  
使用 " **\*.** " 开头，后接多个字符 ,  
比如，后接 do ，会处理所有 以 " **.do** " 结尾的请求。  
**注：  后缀匹配 不以 " / "  开头 。**


## 4.2 请求参数
没有请求参数：  
http://localhost:8080/TestTomcat2/wowowo  
有请求参数：  
http://localhost:8080/TestTomcat2/wowowo<font color=red>?num=10</font>  
**问号** 表示 后面是一些请求参数。num是请求参数名。   
http://localhost:8080/TestTomcat/bmi<font color=red>?weight=60&height=1.9</font>   
**&** 连接多个请求参数。  



# 5. 字符集编码问题

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

# 6. 转发  
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
* 转发可以共享请求对象，响应对象。 **重定向不能共享，因为重定向浏览器会发起两次请求。第一次请求完毕会销毁request 和 response 。**    


# 7. 处理servlet 运行时异常

处理 servlet  运行时异常有两种方法：   

1、 转发到异常处理页面  
绑定异常处理信息到 **request**  
转发到一个异常处理页面  
编写异常处理页面  

2、 交个容器处理  
将异常抛给容器，  throw new ServletException(e)  
在 web.xml 中配置异常处理页面，**<error-page>**  
编写异常处理异常  

通常， **系统异常** (如，数据库连接断了)交给容器处理。   
**应用异常** (如，登录密码错误)一般使用转发来处理。  

# 8. 路径问题  

1. 超链接:  
<a href="delete.do"\>删除<a\>  
2. 表单提交:  
<form action="add.do"\>  
3. 重定向:  
response.sendRedirect("list.do");  
4. 转发：  
request.getRequestDispatcher("listEmp.jsp");   

绝对路径： 以 “/”  开头  
**超链接、表单提交、重定向** 从应用名开始写。   
**转发** 从应用名之后开始写。  

硬编码：路径写死，如：/webApp/jsp/hello.jsp  
注意：  不要把 **应用名** 直接写在路径里，应该使用下面的方法获取部署时的应用名：
```java
String rootDir = request.getContextPath()  
```

相对路径： 不以 “/” 开头  
