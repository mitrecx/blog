---
title: JavaScript笔记4-jQuery 和 AJAX
date: 2018-11-29 10:33:02
tags: ["JavaScript", "jQuery"]
categories: JavaScript
---

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
			var url=encodeURI("http://localhost:8080/ajax01/check.do?name="+username);
			$.ajax({
				type:"POST",
				url:url,
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
