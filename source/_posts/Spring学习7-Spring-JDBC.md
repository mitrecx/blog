---
title: Spring学习7-Spring JDBC
date: 2018-10-23 21:45:37
tags: Spring
categories: Spring
---
# 0. 前言  
最原始的(不考虑ORM框架，不考虑Spring JDBC)，我们用 JDBC 操作数据库通常是以下几步：  
1. 连接数据库：加载驱动，根据url，用户名，密码连接数据库。(初始化连接池)  
2. 获取connection。    
3. 获取statement。  
4. 增删改查，提交。(查询返回ResultSet)  
5. 关闭连接  

通常，我们会编写一个 DBUtils 类来管理连接，然后编写 DAO 封装增删改查方法。  

因为更简便、耦合度低的操作数据库的框架出现，现在人们开发程序很少直接用 JDBC 了。  
本文介绍 Spring 提供的 API JdbcTemplate ，后面再介绍 MyBatis框架。  

<h1 id="id1"> 1. Spring 与 JDBC 整合</h1>  
Spring 提供了：
  * JdbcTemplate  
  * AOP事务管理  
  * 统一异常处理 DataAccessException  

Spring + JDBC 开发步骤：  
1. 搭建开发环境：  
引入各种jar包，包括Spring(Core，AOP，DAO) ,连接池dbcp，JDBC包等。  
2. 编写实体类domain  
3. 编写Dao数据库操作逻辑  
4. applicationContext.xml 中配置，扫描Dao组件，**注入 JdbcTamplate 对象**   

举个例子：  
1、搭建开发环境，略。  
2、 编写 domain 类：  
```java
package cn.mitrecx.domain;
import java.math.BigDecimal;

public class Emp {
	private Integer id;
	private String name;
	private Integer age;
	private BigDecimal salary;
  //get set方法略
}
```
3、 Dao数据库操作：  
```java
package cn.mitrecx.dao;

import java.util.List;
import javax.annotation.Resource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import cn.mitrecx.domain.Emp;
@Repository //扫描到Spring容器
public class EmpDao {
	@Resource
	private JdbcTemplate template;
	/**
	 * 向MySQL数据库emp表中插入一条数据
	 * @author cx
	 * @param emp 实体对象
	 * @time 上午8:43:02
	 */
	public void insert(Emp emp) {
		String sql="insert into emp(name,age,salary) values(?,?,?)";
		Object[] param= {emp.getName(),emp.getAge(),emp.getSalary()};
		//update 可用在 增删改
		template.update(sql, param);
	}

	public void deleteById(int id) {
		String sql="delete from emp where id=?";
		Object[] param= {id};
		template.update(sql, param);
	}
	public void deleteByName(String name) {
		String sql="delete from emp where name=?";
		Object[] param= {name};
		template.update(sql, param);
	}
//查找
	public List<Emp> queryAll(){
		String sql="select * from emp";
		EmpRowMapper empRowMapper=new EmpRowMapper();  //自定义RowMapper
		List<Emp> list=template.query(sql, empRowMapper);
		return list;
	}
}
```
查找时，需要RowMapper参数，用于把数据库字段和domain字段对应：  
```java
package cn.mitrecx.dao;

import java.sql.ResultSet;
import java.sql.SQLException;
import org.springframework.jdbc.core.RowMapper;
import cn.mitrecx.domain.Emp;

public class EmpRowMapper implements RowMapper<Emp> {
	@Override
	public Emp mapRow(ResultSet resultSet, int index) throws SQLException {
		Emp emp=new Emp();
		emp.setId(resultSet.getInt("id"));
		emp.setName(resultSet.getString("name"));
		emp.setAge(resultSet.getInt("age"));
		emp.setSalary(resultSet.getBigDecimal("salary"));
		return emp;
	}

}
```
4、 applicationContext.xml 配置：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context-4.2.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
		http://www.springframework.org/schema/tx
		http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
		http://www.springframework.org/schema/mvc
		http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd">

	<mvc:annotation-driven></mvc:annotation-driven>


	<context:component-scan base-package="cn.mitrecx"/>

	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp" ></property>
	</bean>

	<bean id="template" class="org.springframework.jdbc.core.JdbcTemplate">
		<!-- 注入连接信息 -->
		<property name="dataSource" ref="dbcp"></property>
		<!-- dataSource数据源， 连接池: dbcp,c3p0,proxool -->
	</bean>
	<!-- dbcp连接池 -->
	<bean id="dbcp" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="username" value="root"></property>
		<property name="password" value="123456"></property>
		<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
		<property name="url" value="jdbc:mysql://cherry.mitrecx.cn:3306/DBmitre?useUnicode=true&amp;characterEncoding=utf8"></property>
	</bean>
</beans>
```

测试类：  
```java
package cn.mitrecx.dao;

import java.math.BigDecimal;
import java.util.List;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import cn.mitrecx.domain.Emp;

public class TestDao {
	public static void main(String[] args) {
		ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
		EmpDao empDao=ac.getBean("empDao",EmpDao.class);
//		Emp emp=new Emp();
//		emp.setName("李敖");
//		emp.setAge(23);
//		emp.setSalary(new BigDecimal("5000"));
//		empDao.insert(emp);
//		empDao.deleteByName("李敖");
		List<Emp> list=empDao.queryAll();
		for(Emp emp1:list) {
			System.out.println(emp1.getName()+"  "+emp1.getAge());
		}
	}
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-06_234000.png)  

# 2. Spring MVC 与 JDBC template 结合
Spring MVC 和 JDBC Template 结合，这不是一个新的知识，只是在 Spring MVC 的基础上，加上 [上一节](#id1) 介绍地 JDBC Template 数据库操作。   

直接看一个简单的实例，就清楚怎么写了。  

最近，我也在想，一直写简单的例子，什么时候才能掌握Spring ？  
其实，这应该是一个日积月累的过程。达到一定的量，就会有质的改变。  
能独立做好无数个简单的例子，就已经在深入学习了。  
不要怕一直在做简单的例子，应该考虑如何在最短的时间学会更多的简单的例子。。  

下面演示利用 JDBC Template 向 MySQL 数据库插入一条数据：  

1、servlet 配置文件 web.xml ：    

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd" id="WebApp_ID" version="3.1">
  <display-name>spring11-8-JdbcTemplate</display-name>

  <servlet>
  	<servlet-name>mvc</servlet-name>
  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	<init-param>
  		<param-name>contextConfigLocation</param-name>
  		<param-value>classpath:applicationContext.xml</param-value>
  	</init-param>
  	<load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
  	<servlet-name>mvc</servlet-name>
  	<url-pattern>*.do</url-pattern>
  </servlet-mapping>

</web-app>
```

2、 操作页面 add.jsp :  

```html
<%@page pageEncoding="utf-8" contentType="text/html; charset=utf-8" %>
<html>
	<head>
	</head>
	<body style="font-size: 30px;">
		<form action="addHandle.do" method="post">
			name: <input type="text" name="name"/><br/>
			age: <input type="text" name="age"/><br/>
			salary: <input type="text" name="salary"/><br/>
			<input type="submit" value="提交"/>
		</form>
	</body>
</html>
```

3、 实体类 Emp.java :  

```java
package cn.mitrecx.domain;

import java.math.BigDecimal;

public class Emp {
	private Integer id;
	private String name;
	private Integer age;
	private BigDecimal salary;
  //get set 方法略
}
```

4、 数据库操作类 EmpDao.java :  
DAO类一定要注意 template 成员的值 **是由Spring 容器注入的**。  
**template 在 [applicationContext.xml](#applicationContext) 中配置**。
```java
package cn.mitrecx.dao;

import javax.annotation.Resource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;
import cn.mitrecx.domain.Emp;

@Repository("empDaoID") //扫描到Spring IoC容器，id为empDaoID
public class EmpDao {

	@Resource(name="template") //注入id为template的bean，已在applicationContext.xml中配置
	private JdbcTemplate template;

	public void insert(Emp emp) {
		String sql="insert into emp(name,age,salary) values(?,?,?)";
		Object[] param= {emp.getName(),emp.getAge(),emp.getSalary()};
		//update 可用在 增删改
		template.update(sql, param);
	}

}
```

5、 Spring 配置文件 <span id='applicationContext'>applicationContext.xml</span>:  
**注意组件扫描** 的 base-package 属性，一定要包含所有需要扫描的包。    

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

	<!-- HandlerMapping : 用mvc:annotation-driven包含了多个bean，其中有handlerMapping这个bean -->
	<mvc:annotation-driven/>

	<!-- 扫描controller -->
	<context:component-scan base-package="cn.mitrecx"></context:component-scan>

	<!-- ViewResolver -->
	<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- ViewResolver的前缀后缀属性，用于给controller返回的mav加前后缀 -->
		<property name="prefix" value="/WEB-INF/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>

	<bean id="template" class="org.springframework.jdbc.core.JdbcTemplate">
		<!-- 注入连接信息 -->
		<property name="dataSource" ref="dbcp"></property>
		<!-- dataSource数据源， 连接池: dbcp,c3p0,proxool -->
	</bean>
	<!-- dbcp连接池 -->
	<bean id="dbcp" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="username" value="root"></property>
		<property name="password" value="123456"></property>
		<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
		<property name="url" value="jdbc:mysql://cherry.mitrecx.cn:3306/DBmitre?useUnicode=true&amp;characterEncoding=utf8"></property>
	</bean>
</beans>
```

6、 controller 组件 ：  

```java
package cn.mitrecx.controller;

import java.io.UnsupportedEncodingException;
import java.math.BigDecimal;
import javax.servlet.http.HttpServletRequest;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import cn.mitrecx.dao.EmpDao;
import cn.mitrecx.domain.Emp;

@Controller
public class ViewController {
	@RequestMapping("/add.do")
	public ModelAndView add() {
		ModelAndView mav=new ModelAndView();
		mav.setViewName("add");
		return mav;
	}

	@RequestMapping("/addHandle.do")
	public String addHandle(HttpServletRequest request) {
		try {
			request.setCharacterEncoding("utf-8");
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}

		ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
		EmpDao empDao=ac.getBean("empDaoID",EmpDao.class);
		/*
		 *不能直接 new EmpDao()  
		 *直接new的对象，无法获取到注入到的成员 template(在applicationContext.xml中定义)。
		 *****确切的说，直接new的对象，无法获取到Spring容器注入到 此对象 的成员。
		 */
		//EmpDao empDao=new EmpDao();
		Emp emp=new Emp();
		emp.setName(request.getParameter("name"));
		emp.setAge(Integer.parseInt(request.getParameter("age")));
		emp.setSalary(new BigDecimal(request.getParameter("salary")));

		empDao.insert(emp);

		return "success";
	}
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-08_164355.png)    

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-08_164516.png)  
