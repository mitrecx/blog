---
title: Spring学习2-Spring IoC 管理对象
date: 2018-10-13 03:29:07
tags: Spring
categories: Spring
---
# 1. 管理对象
创建，初始化，释放资源，销毁。  

下载[Spring jar 包](https://repo.spring.io/release/org/springframework/spring/)   

## a. 搭建Spring IoC开发环境  
1. 引入相关jar包  
spring-core-5.0.0.RELEASE  
spring-context-5.0.0.RELEASE  
spring-beans-5.0.0.RELEASE  
spring-expression-5.0.0.RELEASE  
commons-logging  
2. 在src下添加applicationContext.xml文件，内容如下：    

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


</beans>
```

## b. 在applicationContext.xml文件中配置bean，通过ApplicationContext对象取得配置的bean
applicationContext.xml文件 **<font color='#00B2EE'>用构造方法创建bean对象</font>** 配置如下：  
```xml
<!--  相当于 new GregorianCalendar(); 注意id的值唯一-->
<bean id="calendar1" class="java.util.GregorianCalendar">
</bean>
```
java类通过ApplicationContext获取bean  
```java
//实例化Spring容器
ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
//从Spring容器中获取id为calendar1的对象
Calendar c1= ac.getBean("calendar1",Calendar.class);

//以上操作等价于 Calendar c1=new GregorianCalendar();
```
applicationContext.xml文件 **<font color='#00B2EE'>用静态工厂方法创建bean对象</font>** 配置如下(所谓工厂，就是封装了对象场景的细节)：  
```xml
<!--  相当于Calendar.getInstance() ; 注意id的值唯一-->
<bean id="calendar2" class="java.util.GregorianCalendar" factory-method="getInstance">
  <!-- factory-method的值是类中的方法名 -->
</bean>
```
java类通过ApplicationContext获取bean  
```java
//从Spring容器中获取id为calendar2的对象
Calendar c2= ac.getBean("calendar2",Calendar.class);

//以上操作等价于 Calendar c2=Calendar.getInstance();
```
小结：  
不论xml配置中用构造方法创建bean对象，还是用静态工厂方法创建bean对象，在java类里获取bean的方法都是一样的。  
**了解一下**：  
如果我们想要获取对象的属性，而这个属性任然是一个对象。    
比如：  
```java
Date date=c2.getTime();
```
那么如何通过Spring 配置获取呢？获取方法如下：  
applicationContext.xml  
```xml
<bean id="date1" factory-bean="c2" factory-method="getTime">
  <!-- factory-bean的值是Spring容器中现有的对象名 -->
  <!-- factory-method的值是对象中的方法名 -->
</bean>
```
java类  
```java
Date date=ac.getBean("date1",Date.class);
```

**Spring创建bean对象控制**  
1. 控制对象创建方式：<bean\>元素的 **<font color='#FF34B3'>scope属性</font>** 支持 **<font color='#8080f0'>singleton</font>** 和 **<font color='#8080f0'>prototype</font>** 两种创建方式。  
scope的缺省值是singleton，**一般用默认**。  
**<font color='#8080f0'>singleton</font>** 方式：每次 **ac.getBean("id")** 相同id取出的是同一个bean对象 (Spring容器中只实例化一个bean对象)。  
**<font color='#8080f0'>prototype</font>** 方式：每次取出不是同一个bean对象。  

2. 控制对象初始化：<bean\>元素的 **<font color='#FF34B3'>init-method属性</font>** 可以指定bean对象的初始化方法，init-method的值是bean初始化方法名，这样每创建一个bean对象就会调用一次初始化方法 (注：singleton只会初始化一次)。初始化工作放在 **构造方法** 中的话，初始化会在对象创建过程中进行；初始化工作放在 **初始化方法** 中则在对象创建后进行。  

3. 控制对象销毁：<bean\>元素的 **<font color='#FF34B3'>destroy-method属性</font>** 可以指定bean对象的销毁方法。满足下面两个条件才有效：  
---组件(bean)对象为singleton模式  
---调用AbstractApplicationContext容器对象的close()方法   

4. 控制singleton对象创建时机：<bean\>元素 **<font color='#FF34B3'>lazy-init=true</font>** 可以推迟singleton模式的bean对象的创建时间。默认的创建ApplicationContext对象的时候，singleton模式的bean对象就会被创建；lazy-init=true singleton模式的bean对象在实例化时才被创建。  

# 2. 依赖注入(dependency injection)
利用注入设置bean 属性的值。   

<h2 id="1">2.1 set注入</h2>  
举个例子，利用 **set注入** 设置属性的值：  
bean：  
```java
public class User implements Serializable{
	private Integer id;
	private String username;
	private String password;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
}
```
applicationContext.xml 配置：  
```xml
<bean id="user1" class="domain.User">
  <!-- 信息的 set注入，底层调用的是User bean的setter -->
  <property name="id" value="1001"></property>
  <property name="username" value="cx"></property>
  <property name="password" value="123"></property>
</bean>  
```
处理类中调用：  
```java
ApplicationContext ac=new ClassPathXmlApplicationContext("applicationContext.xml");
User u1= ac.getBean("user1",User.class);
//u1的id="1001", username="cx", password="123"
```
说明：  
如果现在需要改u1属性的值，只要修改配置文件即可，不用修改代码了。  
注：  
如果User新加一个属性computer，而Computer有属性cpu, memory，可以使用 **<font color=red>ref</font>** 替代 **<font color=red>value</font>** 注入。  

```xml
<bean id="comp1" class="domain.Computer">
  <property name="cpu" value="corei7"></property>
  <property name="memory" value="8G"></property>
</bean>

<bean id="user1" class="domain.User">
  <!-- 信息的 set注入，底层调用的是User bean的setter -->
  <property name="id" value="1001"></property>
  <property name="username" value="cx"></property>
  <property name="password" value="123"></property>
  <property name="computer" ref="comp1"></property>
</bean>
```
## 2.2 构造器注入
接着上面[set注入的例子](#1) 中的bean，现在做 **构造器注入** ：  
beans:  
```java


public class User implements Serializable{
    private Integer id;
    private String username;
    private String password;
    private Computer computer;

    public User(Integer id,String username,String password,Computer computer) {
  		this.id=id;
  		this.username=username;
  		this.password=password;
          this.computer=computer;
    }
}
```
applicationContext.xml 配置：  
```xml
<!--set注入Computer-->
<bean id="comp2" class="domain.Computer">
  <property name="cpu" value="corei8"></property>
  <property name="memory" value="16G"></property>
</bean>

<!--构造器注入User-->
<bean id="user2" class="domain.User">
  <!-- 构造器注入，底层调用的是User bean的构造方法 -->
  <constructor-arg index="0" value="1002"></constructor-arg>
  <constructor-arg index="1" value="cx"></constructor-arg>
  <constructor-arg index="2" value="123"></constructor-arg>
  <constructor-arg index="3" ref="comp2"></constructor-arg>
</bean>
```
# 3. 总结
简而言之，IoC控制反转就是把对象创建控制权 从处理类手里 交到了Spring手里 ，这一控制权改变建立在 依赖注入 的机制上。  
