---
title: servlet状态管理
date: 2018-09-20 09:18:01
tags:
- servlet
- java
categories: servlet
---
# 状态管理
将浏览器和 web服务器 之间的多次交互作为一个整体来处理。  
并且将多次交互所涉及的数据（状态）保存下来。  

## 1. cookie  
**cookie** 是 服务器临时保存在浏览器端的少量数据。  

**工作原理：**  
1、当 浏览器 访问服务器时，  
服务器可以将少量数据以 **set-cookie消息头** 的方式发送给浏览器，  
浏览器会将这些数据临时保存下来。  
2、当 浏览器 **再次** 访问服务器时，  
会将之前保存的这些数据以 **cookie消息头** 的方式发送给服务器。  

在 response 对象中 **添加 cookie**：  
```java
Cookie c=new Cookie(String name, String value);
response.addCookie(c);
```
从请求对象request中 **读取 cookie**：  
```java
//获取请求对象中的 cookie（可能是多个）
Cookie[] request.getCookies();

String cookie.getName();
String cookie.getValue();
```

## 2. cookie 编码问题
**如果** 中文乱码，那么需要编码( encode ) 放在 cookie 中的汉字。   
比如编码成 "utf-8" (三个字节保存一个汉字)，然后再传输。  

**编码 与 解码**  
```java
//编码：把字符串str 按照 字符集charset 编码
String URLEncoder.encode(String str, String charset);

//解码：把字符串str 按照 字符集charset 解码
String URLDecoder.decode(String str, String charset);
```
注：对添加到 cookie 中的字符串 编码处理，在取出 浏览器发送来的 cookie 时 也要解码处理。  

## 3. cookie 的生存时间
缺省情况下，浏览器会把cookie 保存在内存里，  
只要浏览器不关闭，cookie 就一直存在，  
浏览器关闭，cookie 就会被销毁。  

自己设置 cookie 的生存时间：  
```java
//单位是秒
cookieName.setMaxAge(int seconds);
```
seconds > 0 , 浏览器把 cookie 保存在硬盘上，如果超过了指定时间， cookie 会被删除。  
seconds < 0 , 缺省情况。  
seconds = 0 , cookie 会被立刻删除。一般用于删除已存在的 cookie。  

例如删除已存在的 cookie username：  
```java
//创建一个同名cookie username
Cookie c=new Cookie("username","轩辕荣耀");
//添加到响应对象上，用于发送给浏览器。
//一旦发送给浏览器会把浏览器端的同名cookie替换, 然后删除
c.setMaxAge(0);
response.addCookie(c);
```

## 4. cookie 路径问题
浏览器在向某个地址发请求时，会比较该地址是否匹配 cookie 的路径，如果匹配，那么响应的 cookie 会加载请求报文头里发送出去。  

cookie 的默认路径：  
等于添加该 cookie 的组件 (servlet/jsp) 的路径。  

路径匹配规则：  
请求路径等于cookie 的路径，  
或者请求路径是cookie的子路径。  

## 5. session 会话
session： 服务器端为维护状态而创建的一个特殊的对象。  

**工作原理：**  
浏览器访问服务器时，服务器会创建一个session 对象，  
session对象 有一个唯一的id ，称之为 sessionId。  
服务器会将 sessionId 以 cookie 的方式发送给浏览器。  
浏览器再次访问服务器时，会将 sessionId 以 cookie的方式发送给服务器，服务器端可以利用sessionId找到对应的session对象。  

### 5.1服务端如何获得 session  

方式 1    
```java
HttpSession s=request.getSession(true/false);
```
true:  
先查看 request请求 中有没有 sessionId ，   
如果没有，就创建一个 session对象。  
如果有，就 **依据此 sessionId 查找对应的 session 对象**，如果找到则返回，找不到就创建一个新的session对象。  
false:  
先查看 request请求 中有没有 sessionId ，   
如果没有，返回 null  
如果有，就 **依据此 sessionId 查找对应的 session 对象**，如果找到则返回，找不到就创建一个新的session对象。


方式 2
```java
// 常用写法
HttpSession s=request.getSession();
// request.getSession(true) 的简写形式
// 等价于 request.getSession(true)  
```
### 5.2 session 常用方法
假设 有一个 session 对象 名为 sessionName：  

```java
//获得 session 的 id
String sessionName.getId();

// 绑定数据
sessionName.setAttribute(String name, Object obj);
// 根据绑定名 获得 绑定值
// 该方法可能返回 null
Object sessionName.getAttribute(String name);

// 解除绑定
sessionName.removeAttribute(String name);
```

### 5.3 session 超时  
服务器 会将 空闲时间过长的 session对象 删除。  

缺省的删除时间在 Tomcat 的 server 工程中的 web.xml 中已默认配置。  
```xml
<!-- web.xml 默认 30 minutes -->
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```
可以用编程的方式修改 缺省时间：  
```java
setMaxInActiveInterval(int seconds);
```
### 5.4 删除 session

```java
invalidate();
```
注：  
为什么 不叫 delete 或 drop，而叫 invalidate ？  
因为 容器在实现 invalidate 的时候，并没有删除 session 对象，而是 抹去 session 对象的值，然后重新赋值 再交给用户使用。  

## 6. 登录案例
实现用户名-密码 登录系统机制 详细步骤：  

1、 在数据库中创建一张表(user)，用于存储用户名-密码等信息(隐私信息应当加密)。  

2、 包entity: 创建实体类 (User)。   

3、 包dao: 创建 DBUtil.java 连接数据库工具类，创建 UserDao.java 提供 数据库操作。  

4、 包web: ActionServlet.java   

5、 WebContent 目录: login.jsp 登录页面，success.jsp 登录成功页面。  

6、 **session 验证：**   
登录成功后，将一些数据绑定到 session 对象上，  
比如，session.setAttribute("user",user);  
对于需要保护的资源(success.jsp) , 添加session验证代码：  
```java
//ActionServlet.java 代码:
//登录成功，将一些数据绑定到session对象上
HttpSession session=request.getSession();
session.setAttribute("user", user);
response.sendRedirect("success.jsp");
//登录失败/不登录 不绑定 数据到session
```
success.jsp 代码：  
```java
Object obj=session.getAttribute("user");
// 如果没有登录
if(obj==null){
  response.sendRedirect("login.jsp");
}
```
注：  
登录成功，就绑定一些数据到 session 对象，登录失败则不绑定。  
所以，在登录成功的页面 需要 对session对象 验证是否有 绑定数据，  
如果有就显示登录成功的页面，如果没有 就返回登录页面。  
