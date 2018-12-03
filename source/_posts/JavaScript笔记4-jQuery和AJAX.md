---
title: JavaScript笔记4-jQuery 和 AJAX
date: 2018-11-29 10:33:02
tags: ["JavaScript", "jQuery"]
categories: JavaScript
---
# 1 jQuery 和 AJAX
接着 [上一节( JavaScript笔记3-AJAX )的例子](https://mitrecx.github.io/2018/11/20/JavaScript%E7%AC%94%E8%AE%B03-AJAX/#head4)  

目录结构如下：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-29_155751.png)  

其他文件不变，增加 demojquery.jsp ，此文件实现的功能和 demo2.jsp 相同。  
( demojquery.jsp 只是用 jQuery 完成 demo2.jsp 的功能。)  

demojquery.jsp 内容 如下：  
```html
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8"%>
<html>
<head>
	<script type="text/javascript" src="jquery-3.3.1.min.js"></script>
	<script type="text/javascript">
		// $()页面加载完毕后执行
		$(function(){
			$("#szid").click(function(){
				var url=encodeURI("http://localhost:8080/ajax01/demo1.do");
				$.ajax({
					url:url,
					type:"POST",
					success:function(result){
						// result就是服务器返回的内容 等价于 xhr.responseText
						$("#msg").html(result);
					}
				});
			})
		})

		// 检查用户名是否可用
		function checkName(){
			var username=$("#username").val();
			//console.log(username);
			$("#name_msg").html("检查中.....");
//			var url=encodeURI("http://localhost:8080/ajax01/check.do?name="+username);
			var url=encodeURI("http://localhost:8080/ajax01/check.do");
			$.ajax({
				type:"POST",
				url:url,
				data:{"name":username},
				success:function(result){
					$("#name_msg").html(result);
				}
			});
		}
	</script>
</head>

<body style="font-size: 30px;">
	<a href="#" id="szid">查看上证指数</a>
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
			<td>大海你真美</td>
			<td>2018-11-21</td>
		</tr>
	</table>
	<input type="text" id="username" onblur="checkName();" />
	<span id="name_msg"></span>
</body>
</html>
```

常用的 ajax 配置参数：  
```JavaScript
$.ajax({
	url: 请求地址,
	type: 请求类型(POST|GET),
	data: 提交的数据,
	async: 同步或异步(true)处理,
	dataType: 预期服务器返回的数据类型,
	success: 成功回调函数,
	error: 失败回调函数,
	beforeSend: 请求发送前回调函数
});
```
# 2 servlet 返回 JSON  
实现 网页列表显示 功能。  

NoteListServlet2.java ，这个 servlet 返回的字符串是 由 **JSONArray** 类型 toString 得到的。  
NoteListServlet2 返回的本质是一个 **JSON数组**。  

(NoteListServlet.java 仅做参考，内容和NoteListServlet2.java基本一样。 它返回的字符串是 由 **JSONObject** 类型 toString 得到的。  
NoteListServlet 返回的本质是一个 **JSON对象**。)  

```java
package cn.mitrecx.servlet;

public class NoteListServlet2 extends HttpServlet {
  @Override
  protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html;charset=utf-8");
    //将笔记以 JSON 格式输出
    /*
     * 返回 一个 json 对象
     * json:  {id:"01",name:"JavaScript学习笔记"}
     */
    Note note = new Note();
    note.setId("01");
    note.setName("JavaScript学习笔记");
    JSONObject jsonObject=JSONObject.fromObject(note);
    String jsonObjectString=jsonObject.toString();
    //System.out.println(jsonObjectString);
    PrintWriter out=response.getWriter();
    /*
     * 返回 json 数组
     * [{"id":"01","name":"JavaScript学习笔记"},{"id":"02","name":"JSON作为返回数据类型"}]
     */
    Note note2=new Note();
    note2.setId("02");
    note2.setName("JSON作为返回数据类型");
    List<Note> list=new ArrayList<Note>();
    list.add(note);
    list.add(note2);
    JSONArray jsonArray=JSONArray.fromObject(list);
    String jsonArrayString=jsonArray.toString();
    //out.println(jsonObjectString); //NoteListServlet 返回
    out.println(jsonArrayString); //NoteListServlet2 返回

    out.flush();
    out.close();
  }
}
```
web.xml  
```xml
<servlet>
	<servlet-name>ajax-json</servlet-name>
	<servlet-class>cn.mitrecx.servlet.NoteListServlet</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>ajax-json</servlet-name>
	<url-pattern>/ajax.do</url-pattern>
</servlet-mapping>
<servlet>
	<servlet-name>ajax-json2</servlet-name>
	<servlet-class>cn.mitrecx.servlet.NoteListServlet2</servlet-class>
</servlet>
<servlet-mapping>
	<servlet-name>ajax-json2</servlet-name>
	<url-pattern>/ajax2.do</url-pattern>
</servlet-mapping>
```
demo.html ，这才是重点。  
重点是 AJAX请求返回的数据的处理。  
**<font color=red>JSON 和 JSON数组 ，dataType 都是 JSON。但是JSON对象要按对象处理，数组要按数组处理。</font>**  

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="content-type" content="text/html;charset=utf-8" >
    <script src="jquery-3.3.1.min.js"></script>
    <script type="text/javascript">
      $(function(){
        $("#noteListBtn").click(function(){
          $("#noteList").empty();
          $.ajax({
            type: "post",
            url: "http://localhost:8080/ajax-json/ajax.do",
            dataType: "json",
            success: function(result){
              // Object.keys(result).length 获取JSON对象的长度
              console.log(Object.keys(result).length);
              var id=result.id;
              var name=result.name;
              // 拼成 dom对象
              var dom_li="<li>"+name+"</li>";
              var $li=$(dom_li);//转为jQuery对象
              //$li.data("id",id);
              $("#noteList").append($li);
          	}
          });
        });
      })
    </script>

    <script type="text/javascript">
    $(function(){
      $("#noteListBtn2").click(function(){
        $("#noteList2").empty();
        $.ajax({
          type: "post",
          url: "http://localhost:8080/ajax-json/ajax2.do",
          dataType: "json",
          success: function(result){
            // 获取数组(json数组)的长度 result.length
            console.log(result.length);
            //result 就是 服务器返回的json 串
            for(var i=0; i<result.length; ++i){
              console.log(i);
              var id=result[i].id;
              var name=result[i].name;
              // 拼成 dom对象
              var dom_li="<li>"+name+"</li>";
              var $li=$(dom_li);//转为jQuery对象
              //$li.data("id",id);
              $("#noteList2").append($li);
          	}
          }
        });
      });
    })
    </script>
  </head>
  <body>
    <input id="noteListBtn" type="button" value="显示 NoteList"/>
    <hr/>
    <ul id="noteList">
    </ul><br/>
    <input id="noteListBtn2" type="button" value="显示 NoteList对象数组"/>
    <ul id="noteList2">
    </ul>
  </body>
</html>
```

结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-12-01_164516.png)  

# 3 SpringMVC 返回 JSON
开发步骤：  
1、 DispatchServlet  
2、 HandlerMapping  
3、 Controller (处理完返回一些数据)  
4、 调用 spring 自带包jackson 将 Controller 返回结果转成 json 输出  

Controller:  
```java
package cn.mitrecx.controller;
import java.util.ArrayList;
import java.util.List;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import cn.mitrecx.domain.Note;
@Controller
public class NoteListController {
  @RequestMapping("/springjson.do")
  @ResponseBody //使 返回结果 转为 json. 有此标记就不会走ViewResolver了
  public List<Note> execute() {
    List<Note> list=new ArrayList<Note>();
    Note note1=new Note("01","SpringMVC 返回json");
    list.add(note1);
    Note note2=new Note("02","ajax是一种前端技术");
    list.add(note2);
    return list;//直接return，由Jackson 自动转成json
  }

  @RequestMapping("/springjson2.do")
  @ResponseBody
  public Note show() {
    Note note=new Note("22","json是一种数据格式");
    //domain类，List，Map都能转 JSON
    return note;
  }
}
```
domain类:  
```java
package cn.mitrecx.domain;
import java.io.Serializable;
public class Note implements Serializable{
  private String id;
  private String name;
  public Note() {
  }
  public Note(String id,String name) {
    this.id=id;
    this.name=name;
  }
  public String getId() {
    return id;
  }
  public void setId(String id) {
    this.id = id;
  }
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
}
```
web.xml  
```xml
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</init-param>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>*.do</url-pattern>
</servlet-mapping>
```
applicationContext:  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
		http://www.springframework.org/schema/util
		http://www.springframework.org/schema/util/spring-util-4.3.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

	<!-- HandlerMapping -->
	<mvc:annotation-driven/>

	<!-- controller -->
	<context:component-scan base-package="cn.mitrecx.controller"></context:component-scan>
</beans>
```
**demo.xml 和 上一例一样，只要改一下URL**。  

结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-12-01_195618.png)  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-12-01_201506.png)  

ajax 学习 完！
