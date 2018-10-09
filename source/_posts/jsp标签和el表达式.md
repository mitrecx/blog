---
title: jsp标签和el表达式
date: 2018-10-10 20:58:35
tags: jsp
categories: jsp
---
# jsp标签和el表达式

## 1. jsp标签
jsp标签用来替换jsp文件中的java代码。    
因为直接在jsp文件中写java代码，不方便维护，  
所以sun制定了 **<font color=red>jsp标签技术规范</font>**。  

可以将jsp标签看做一个占位符，容器遇到jsp标签，会依据标签名找到对应的标签类，然后执行标签类中的代码。  
**简而言之，jsp标签用来代替jsp文件中的java代码**   

**使用jsp标签的好处：**  
1、 jsp文件方便维护  
2、 方便代码的复用  

**jstl ：java标准标签库( java standard tag lib ),** apache 开发的一套标签库，捐献给sun，sun将其命名为jstl。

**如何使用 jstl**：  
1、 将jstl相关的jar文件拷贝到 WEB-INF/lib 目录下。   
注：如果使用javaEE 5.0(包含了jstl相关的jar文件)，一般不用拷贝，不绝对，和Tomcat版本有关。      
2、 使用 **<font color='#FF34B3'>taglib</font>**  指令导入相应的标签。  

**jstl 几个常用的标签：**   
1、 if  
uri取自standard.jar的META-INF目录中的c.tld(或c-rt.tld)的uri  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/jsp-taglib1.png)  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/jsp-taglib2.png)
c 是命名空间的前缀(别名)
```html
<%@taglib uri="http://java.sun.com/jstl/core" prefix="c"%>

<c:if test="" var="" scope="">  
<!-- test属性为true时，容器会执行if标签体，test属性可以使用el表达式-->
<!-- var属性：指定绑定名称（绑定结果是test结果） -->
<!-- scope属性：指定绑定范围，page，request，session，application-->
</c:if>
```
2、choose  
```html
<c:choose>
  <c:when test="">
    <!--when可以出现一次或多次，当test属性为true时，执行when下的语句-->
  </c:when>
  <c:otherwise>
    <!--otherwise可以出现0次或1次，表示其他情况-->
  </c:otherwise>
</c:choose>
```
3、forEach  
```html
<!--用来遍历集合或数组-->
<c:forEach items="" var="" varStatus="">
  <!--items属性指定要遍历的集合或数组-->
  <!--var属性指定一个绑定名，绑定范围固定是pageCont-->
</c:forEach>
<!--注：每次从集合或数组中取一个元素，然后绑定到pageContext上，绑定名由var属性指定-->
```

## 2. el表达式
el (Expression Language) 表达式是一套简单的运算规则，用于给jsp标签的属性赋值，也可以直接输出。  

**el 表达式的使用：**  
1、 **<font color='#8080f0'>访问 bean 的属性</font>**  
* 方式一：**<font color='#8080f0'>${user.name}</font>**  
容器会依次从 pageContext，request，session，application 中查找（getAttribute) 绑定名为 **"user" 的对象** ，找到该对象后，调用该对象的 **"getName" 方法** ，然后输出。  
可以使用 pageScope, requestScope,sessionScope,application 指定查找范围，如，${pageScope.user.name}  
* 方式二：**<font color='#8080f0'>${user["name"]}</font>**  
[ ]里可以是一个绑定名，也可以是从0开始的下标（访问数组）  

2、 **<font color='#008B00'>进行一些简单的运算，运算结果可以给jsp标签赋值，也可以直接输出</font>**  
* 算术运算 " **+, -, \*, /, %** "  
**<font color='#008B00'>如，${1+1}, ${"3"+"4"}</font>**
* 关系运算 " **>, >=, <, <=, ==, !=** "  
**<font color='#008B00'>如，${2>1}, ${user.name=="cx"}</font>**  
* 逻辑运算 " **&&, ||, !** "  
**<font color='#008B00'>如，${1<2 || 3<4} </font>**  
* empty运算, 判断集合是否为空，或字符串是否是一个空字符串  
**<font color='#008B00'>如，${empty str}</font>** str为空返回true，不空为false  

3、 **<font color='#00B2EE'>读取请求参数值，了解一下 (jsp负责展示数据，不适合处理请求)</font>**  
* **<font color='#00B2EE'>${param.username}</font>**  
等价于 request.getParameter("username")  
* **<font color='#00B2EE'>${paramValues.friends}</font>**  
等价于request.getParameterValues("friends")  
