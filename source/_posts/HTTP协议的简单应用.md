---
title: HTTP协议的简单应用
date: 2018-09-01 21:27:44
tags: HTTP协议
categories: HTTP协议
---
# 1. 什么是HTTP协议

由 [w3c](https://www.w3.org/) 制定的网络应用层协议，
规定了 __浏览器与web服务器 __ 之间如何通信，以及数据包的结构。  
**通信过程：**  
step1. 建立连接  
step2. 发送请求  
step3. 发送相应  
step4. 关闭连接  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/http-%E7%AE%80%E5%8D%95%E5%BA%94%E7%94%A81.png)  

** 特点：**  
一次请求，一次连接。  
即如果浏览器需要发送新的请求，就需要建立新的连接。  
这样设计的优点是，服务器可以利用 **有限的连接** 为尽可能多的 **请求** 服务。  


# 2. HTTP数据包格式
1. 请求数据包  
请求行  
消息头  
实体内容  

2. 相应数据包  
状态行  
消息头  
实体内容  

利用 eclipse 自带的抓包工具 抓个包：  
打开 window -> show view -> other -> debug -> tcp/ip monitor  
右击空白处-> properties -> add  
比如简单配置如下  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/eclipse-tcpIpMonitor.png)  
OK 完了 start。
现在访问  
```sh
#以下路径
# http://localhost:8888/TestTomcat2/bmi?height=1.7&weight=100
```

注意端口是8888，这个monitor就是一个代理服务器，它会转发到Tomcat 监听的8080端口。  
[源代码](https://github.com/mitrecx/TestTomcat)  

现在看看 monitor 转发了什么(show header)  
**Request:**
```sh
#请求行
#注：GET请求会把请求参数放在路径的后面
GET /TestTomcat2/bmi?height=1.7&weight=100 HTTP/1.1  
# 键值对 用冒号: 隔开
Host: localhost:8080      

#消息头
Connection: keep-alive  
Upgrade-Insecure-Requests: 1  
#浏览器信息
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)AppleWebKit/537.36 (KHTML, like Gecko)Chrome/68.0.3440.84 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8  
Accept-Encoding: gzip, deflate, br  
Accept-Language: en,zh;q=0.9,lb;q=0.8,zh-CN;q=0.7,en-US;q=0.6  

#实体内容
#这是一个GET请求，实体内容没有数据！
#只有POST请求，才有内容
```

**Response:**   
```sh
#状态行
HTTP/1.1 200

#消息头
#Content-Type服务器返回的数据类型
Content-Type: text/html;charset=ISO-8859-1  
Content-Length: 39  
Date: Wed, 08 Aug 2018 16:31:22 GMT

#实体内容
34.602076124567475
<h1>my pride</h1>
#浏览器解析其中的数据，生成相应的页面
```


# 3. HTTP的两种请求方式
## GET请求
以下情况，浏览器发送 get 请求  
1. 直接输入url
2. 点击链接
3. 表单默认提交方式

```KHTML
<!--表单的 提交方式method 默认就是get -->
<form method="get">
</form>
```
GET请求特点：  
1. 请求参数放在请求资源路径的后面
2. 最多只能提交2k字节
3. 不安全

## POST请求  
以下情况，浏览器发送 post 请求
1. 设置表单 method="post"   

```xhtml
<html>
  <head>
    <!-- 模拟content-type消息头 -->
    <meta http-equiv="content-type" content="text/html;charset=utf-8">
  </head>
  <body style="font-size:30px;">
    <form action="TestTomcat2" method="post">
      <fieldset>
        <legend>欢迎</legend>
        身高：<input name="height"/>
        体重：<input name="weight"/>
        <input type="submit" value="确定"/>
      </fieldset>
    </form>
  </body>
</html>
```
```sh
# 打开URL
http://localhost:8888/TestTomcat2
# 填好表单，点击提交
# 将会自动跳转到以下URL，请求参数不会显示出来
http://localhost:8888/TestTomcat2/bmi
```

post 请求特点：
1. 不会把请求参数显示在URL中
2. 相对安全  （注：不会对请求参数加密）   
