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
2. **<font color='#bf00ff'>XMLHttpRequest 发请求</font>**   
3. Tomcat服务器 处理请求返回响应结果给浏览器    
4. **<font color='#bf00ff'>XMLHttpRequest 接收响应</font>**  
5. 浏览器显示结果   

AJAX 局部刷新，异步处理 **提高了程序交互效率**。  

AJAX 技术：  
* 基于 JS 发请求和响应处理。  
* **以 XMLHttpRequest对象 为核心**。  
* 涉及 HTML，CSS 页面渲染技术。  
* 涉及 XML，JSON 等数据交互格式。  

----

举一个例子：看看 普通交互模式 和 AJAX交互模式 的区别。     

**普通交互模式**:  
web.xml  
```XML
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  	<display-name>ajax01</display-name>
	<servlet>
		<servlet-name>demo1</servlet-name>
		<servlet-class>cn.mitrecx.DemoServlet</servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name>demo1</servlet-name>
		<url-pattern>/demo1.do</url-pattern>
	</servlet-mapping>
</web-app>
```
DemoServlet.java  
```java
package cn.mitrecx;

public class DemoServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    request.setCharacterEncoding("utf-8");
    response.setContentType("text/html;charset=utf-8");
    PrintWriter out = response.getWriter();
    out.println("上证指数:2645.43");
    out.flush();
    out.close();
  }
}

```
demo1.jsp  
```HTML
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8" %>
<html>
	<body>
		<a href="demo1.do" >查看上证指数</a>
		<hr/>
		<table>
			<tr>
				<td>股市新闻</td>
				<td>时间</td>
			</tr>
			<tr>
				<td>新一代股神mitre</td>
				<td>2018-11-22</td>
			</tr>
			<tr>
				<td>股市肿么了</td>
				<td>2018-11-21</td>
			</tr>			
			<tr>
				<td>大海你真美</td>
				<td>2018-11-21</td>
			</tr>			
		</table>
	</body>
</html>
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_225259.png)   

**AJAX交互模式**:  
web.xml 和 DemoServlet.java 与普通交互模式相同。  

jsp文件做以区分，命名为 demo2.jsp，内容入下：  
```JavaScript
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8"%>
<html>
<head>
<script type="text/javascript">
	//创建 XMLHttpRequest对象
	function getXhr() {
		var xhr
		if (window.XMLHttpRequest) {
			xhr = new XMLHttpRequest();
		} else {//老版IE
			xhr = new ActiveXObject("Microsoft.XMLHTTP");
		}
		//alert(xhr);
		return xhr;
	}

	function sendRequest() {
		var xhr = getXhr();//获取XMLHttpRequest对象
		xhr.open("get", "demo1.do");//创建一个HTTP请求
		//注册一个回调函数
		xhr.onreadystatechange = function() {
			//readyState==4 表示请求处理完毕
			if (xhr.readyState == 4) {
				//获取服务器返回的信息
				var msg = xhr.responseText;
				//将消息放到span中显示
				document.getElementById("msg").innerHTML = msg;
			}
		};
		xhr.send(null);//发送HTTP请求
	}
</script>
</head>

<body style="font-size: 30px;">
	<a href="#" onclick="sendRequest();">查看上证指数</a>
	<span id="msg"></span>
	<hr />
	<table>
		<tr>
			<td>股市新闻</td>
			<td>时间</td>
		</tr>
		<tr>
			<td>新一代股神mitre</td>
			<td>2018-11-22</td>
		</tr>
		<tr>
			<td>股市肿么了</td>
			<td>2018-11-21</td>
		</tr>
		<tr>
			<td>大海你真美</td>
			<td>2018-11-21</td>
		</tr>
	</table>
</body>
</html>
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_225715.png)  
