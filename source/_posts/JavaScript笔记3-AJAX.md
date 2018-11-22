---
title: JavaScript笔记3-AJAX
date: 2018-11-20 21:41:34
tags: JavaScript
categories: JavaScript
---
**AJAX** = Asynchronous JavaScript and XML（**异步的 JavaScript 和 XML**）。  

xml 和 json ：
```XML
<person>
  <name>mitre</mitre>
  <age>24</age>
</person>
```
```JSON
{"name":"mitre", "age":"24"}
```

AJAX 通过 XMLHttpRequest 对象发请求：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-21_235128.png)  

普通交互模式（特点：全页面刷新 + 请求与响应同步处理(一个请求发送必须等到对应的相应回来才能再发下一个请求)）：   
1. 浏览器发请求   
2. Tomcat服务器 处理请求返回响应结果给浏览器   
3. 浏览器显示结果   

AJAX交互模式（特点：局部刷新 + 请求与响应异步处理）：   
1. 浏览器发请求   
2. XMLHttpRequest 发请求   
3. Tomcat服务器 处理请求返回响应结果给浏览器    
4. XMLHttpRequest 接收响应  
5. 浏览器显示结果   

AJAX 局部刷新，异步处理 **提高了程序交互效率**。  

50:00
