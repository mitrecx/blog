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

# 1 普通交互模式  

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

# 2 AJAX交互模式    
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

# 3 XMLHttpRequest 对象  

XMLHttpRequest 对象目前还没有完全标准化。

**XMLHttpRequest 对象提供了基于 HTTP协议 的交互方法。包括发送 POST 和 HEAD 请求和普通的 GET 请求。**  
XMLHttpRequest 可以同步或异步地返回 Web服务器 的响应，并能够以文本或 DOM文档 的形式返回内容。  

XMLHttpRequest 并不限于和 XML 文档一起使用：**它可以接收任何形式的文本文档**。  

## 3.1 属性

### 1 readyState  
readyState 表示 HTTP 请求的状态。  
当一个 XMLHttpRequest 初次创建时，readyState = 0 ；接收到完整的 HTTP 响应时，这个值增加到 4。  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-29_091814.png)  
每次这个属性的值增加的时候，都会 **触发 onreadystatechange 事件句柄**。  

### 2 responseText
responseText属性的值 是到目前为止 **客户端接收到的响应体**（不包括头部）。  

<code>readyState &lt; 3 </code>，responseText属性值 是一个空字符串。  
<code>readyState = 3 </code>，responseText属性值 是目前已经接收的响应部分。  
<code>readyState = 4 </code>，responseText属性值 是完整的响应体。  

### 3 responseXML
功能和 responseText 类似。  
对请求的响应，解析为 XML 并作为 Document 对象返回。  

### 4 status
由服务器返回的 HTTP 状态码。  
如 200 表示成功，404 表示 "Not Found" 错误。  
当 readyState 小于 3 的时候读取这一属性会导致一个异常。

## 3.2 事件句柄
### onreadystatechange  
每次 **readyState属性** 改变的时候调用的事件句柄函数。  
当 readyState 为 3 时，它也可能调用多次。  

## 3.3 方法
### 1 open()  
```java
open(method, url, async, username, password)
// method 参数是用于请求的 HTTP 方法。值包括 GET、POST 和 HEAD
//async 参数指示请求是否异步执行，默认不填或 true 请求为异步。 false为同步。
//username 和 password 参数是可选的，为 url 所需的授权提供认证资格。
```
**初始化 HTTP 请求参数**。  
例如 URL 和 请求方法，但是并不发送请求。  

**<font color=red>open() 会把 readyState 设置为 1</font>**。


### 2 send()  
```java
send(body)
```
**发送 HTTP 请求**。  
如果 readyState 不是 1，send() 抛出一个异常。否则，它发送一个 HTTP 请求(POST 或 PUT、GET等)。  
该请求由以下几部分组成：  
1. 调用 open() 时指定的 HTTP 方法、URL 以及认证资格（如果有的话）。  
2. 调用 setRequestHeader() 时指定的请求头部（如果有的话）。  
3. 传递给这个方法的 body 参数。  

一旦请求发布了，**<font color=red>send() 会把 readyState 设置为 2</font>**，并触发 onreadystatechange 事件句柄。


POST请求 的请求参数 需要放在 send() 中发送:  
```java
// post请求 http://localhost:8080/ajax01/check.do?name=cherry
xhr.open("POST","/ajax01/check.do");
// setRequestHeader 和 事件句柄 处理略
xhr.send("name="+"cherry");
```
而 GET 请求的请求参数放在 open中：  
```java  
//get请求 http://localhost:8080/ajax01/check.do?name=cherry
xhr.open("GET","/ajax01/check.do?name="+"cherry");
// 事件句柄 处理略
xhr.send(null);
```


### 3 setRequestHeader()
POST请求 必须给 HTTP协议 设置请求头参数(不设置服务端将取不到请求参数)。  
```java
xhr.setRequestHeader("content-type","application/x-www-form-urlencoded");
```

### 4 abort()
取消当前响应，关闭连接并且结束任何未决的网络活动。  

abort() 会把 readyState属性值 置为 0 ，并且取消所有未决的网络活动。  
如果请求用了太长时间，而且响应不再必要的时候，可以调用 abort()。  

<h1 id="head4">4 附例</h1>

在上例基础上， 修改demo2.jsp 实现 "检查用户名是否可用" 功能 。([查看本例的 jQuery实现](https://mitrecx.github.io/2018/11/29/JavaScript%E7%AC%94%E8%AE%B04-jQuery%E5%92%8CAJAX/))  
```html
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
	<script type="text/javascript">
		//检查用户名是否可用
		function checkName(){
			var name=document.getElementById("username").value;
			var xhr=getXhr();
			/* get请求 */
			//xhr.open("GET","/ajax01/check.do?name="+name);
			//xhr.send(null);
			/* post请求 */
			xhr.open("POST","/ajax01/check.do");
			// POST请求必须给HTTP协议设置请求头参数(不设置服务端将取不到请求参数)
			xhr.setRequestHeader("content-type","application/x-www-form-urlencoded");
			//注册回调函数
			xhr.onreadystatechange=function(){
				//如果请求处理完毕且响应正常
				if(xhr.readyState==4 && xhr.status==200){
					var msg=xhr.responseText;
					document.getElementById("name_msg").innerHTML=msg;
				}
			}
			xhr.send("name="+name);		
			document.getElementById("name_msg").innerHTML="正在检测..";
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
	<input type="text" id="username" onblur="checkName();"/>
	<span id="name_msg"></span>
</body>
</html>
```
CheckServlet.java  
```java
package cn.mitrecx;

public class CheckServlet extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    try {
      Thread.sleep(5000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    request.setCharacterEncoding("utf-8");
    response.setContentType("text/html;charset=utf-8");
    PrintWriter out=response.getWriter();
    String name=request.getParameter("name");
    System.out.println(name);
    //检查 用户名 是否可用
    if("mitre".equals(name)) {
      out.println("用户名已被占用");
    }else {
      out.println("用户名可用");
    }
    out.flush();
    out.close();
  }
}
```
web.xml  
```xml
<servlet>
  <servlet-name>checkServlet</servlet-name>
  <servlet-class>cn.mitrecx.CheckServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>checkServlet</servlet-name>
  <url-pattern>/check.do</url-pattern>
</servlet-mapping>
```
先测试 servlet :  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-28_224552.png)  

程序运行结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-29_085332.png)    
