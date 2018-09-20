---
title: jsp学习
date: 2018-09-01 21:32:55
tags:
- java
- servlet
- jsp
categories:
- servlet
- java
---
# 1. jsp 简介
jsp 是 sun 公司制定的一种 **服务器端动态页面技术** 规范。  

jsp 文件以 “.jsp” 结尾，文件内容主要是 HTML 和 少量 java 代码。**容器会将 jsp 文件自动转换成一个 servlet ，然后执行**。   

我们用 (PrintWriter)out.println 可以动态地生成页面，那么为什么还需要 jsp ？  
原因有二：  
1. out.println 只适用于生成简单的页面。生成复杂的页面，修改将会十分困难。  
2. jsp 将页面与 java控制逻辑代码(servlet) 分开，专注 **动态生成页面**。简洁，简单，功能强大。  

# 2. 如何写一个 jsp
step1. 创建 ".jsp" 文件  

step2. 添加以下内容：   
1、 HTML(css, js) 直接写  

2、 java 代码：  

① java 代码片段  
 <% java 语句; %>

② jsp 表达式  
<%= java表达式 %> 等价于 out.print(java对象);  
jsp表达式的唯一的作用是，不用写print 语句了。。。  

3、 隐含对象(9个)  

在 jsp 文件中可以直接使用的对象：  
out  
request  
response  
session

4、 指令  
指令：通知容器，在将 jsp 文件转化成 servlet 类时，做一些额外的处理，比如 导包。  

指令的语法：  
<%@ 指令名称 属性=属性值 %>  

**page 指令：**  
import 属性 用于导包  
例如：
```jsp
<%@page import = "java.util.*" %>
```
contentType 属性 response.setContentType() 的内容    
pageEncoding 属性 jsp 文件的编码  
例如：
```jsp
<%@page contentType="text/html;charset=utf-8" pageEncoding="utf-8" %>
```

**include指令:**   
将 jsp 文件转换成 servlet 类时，将 file 属性 指定的文件的内容插入到所在位置：  
```jsp
<%@include file="header.jsp" %>
```


# 3. jsp 是如何执行的  

1. **容器将 jsp 文件转换成一个 servlet 类**  

HTML（css, js） --->  放在service 方法中，用out.write 输出。  

<% java 语句; %>  ---> 放在service 方法中，原封不动。   

<%= java表达式 %> ---> 放在service 方法中，out.print(java表达式) 输出。  

2. ** 容器调用 servlet **  

可以看出，jsp 最终还是转换成 java 代码了。  
如： hello.jsp 执行以后 生成 hello_jsp.java  
