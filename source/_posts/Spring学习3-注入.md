---
title: Spring学习3-注入
date: 2018-10-13 21:18:24
tags: Spring
categories: Spring
---
# 1. 各种类型注入的配置方式
给bean的属性注入值，这些配置方式都是固定的，不难理解。  
所以直接给出一个例子展示各种类型的注入方法。  
bean：  
```java
package domain;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

public class Person {
	private String name;
	private int age;
	private List<String> friends;
	private Set<String> interests;
	private Map<String,String> books;
	private Properties db;

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public List<String> getFriends() {
		return friends;
	}
	public void setFriends(List<String> friends) {
		this.friends = friends;
	}
	public Set<String> getInterests() {
		return interests;
	}
	public void setInterests(Set<String> interests) {
		this.interests = interests;
	}
	public Map<String, String> getBooks() {
		return books;
	}
	public void setBooks(Map<String, String> books) {
		this.books = books;
	}
	public Properties getDb() {
		return db;
	}
	public void setDb(Properties db) {
		this.db = db;
	}
}
```
## 1.1 注入字符串，数值  
字符串注入[上一篇博客](https://mitrecx.github.io/2018/10/13/2-Spring-IoC-%E7%AE%A1%E7%90%86%E5%AF%B9%E8%B1%A1/)已经涉及到了。  
数值注入的话，Spring会在内部做类型转换，所以我们可以为数值类型(如，int)注入可以转换为数值字符串。  
```xml
<bean id="p1" class="domain.Person">

  <property name="name" value="cx"></property>
  <property name="age" value="24"></property>

</bean>
```
注：如果想要给name属性注入空值null  
```xml
  <property name="name">
    <null/>
  </property>
```

## 1.2 注入 List 和 Set

```xml
<bean id="p1" class="domain.Person">

  <property name="friends">
    <list>
      <value>汤姆</value>
      <value>杰克</value>
    </list>
  </property>

  <property name="interests">
    <set>
      <value>足球</value>
      <value>篮球</value>
      <value>乒乓球</value>
    </set>
  </property>

</bean>
```

## 1.3 注入Map

```xml
<bean id="p1" class="domain.Person">

  <property name="books">
    <map>
      <entry key="朱光潜" value="谈美"></entry>
      <entry key="亚当斯密" value="国富论"></entry>
    </map>
  </property>

</bean>
```

## 1.4 注入Property

```xml
<bean id="p1" class="domain.Person">

  <property name="db">
    <props>
      <prop key="username">root</prop>
      <prop key="password">123</prop>
      <prop key="driver">com.mysql.jdbc.Driver</prop>
    </props>
  </property>

</bean>
```

## 1.5 使用表达式注入

\#{表达式}  
\#{id名.属性} 或 \#{id名.key}  
如果是对象属性的话，需要类有getter。  


## 1.6 使用 util:
使用 **util:集合类型** 可以把集合的配置放在 **<beans\>** 标签下。  
当别的 **<bean\>** 需要时，可以直接调用集合。  

使用 **util:** 需要引入命名空间：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:util="http://www.springframework.org/schema/util"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
  http://www.springframework.org/schema/util
  http://www.springframework.org/schema/util/spring-util-4.3.xsd">

<util:list id="list1">
  <value>汤姆</value>
  <value>杰克</value>
</util:list>

<!-- 调用集合对象list1，和前面1.2中的配置等价 -->
<bean id="p1" class="domain.Person">
  <property name="friends" ref="list1">
  </property>
</bean>

</beans>
```

注：  
**util:集合类型** 包括：  
util:list  
util:set  
util:map  
util:properties  

<util: xxx\> 和 <xxx\> 的子标签是一样的。  
当集合需要 **重复利用** 时，**util:集合类型** 显然是更好的选择。  

值得注意的是，<util: properties\> 可以读取文件：  
比如，在src目录下有一个db.properties文件：  
```shell
# key = value
username = root
password = 123
driver = com.mysql.jdbc.Driver
```
在applicationContext.xml 文件中用 <util: properties\> 的 location 属性读取：  
```xml
<util:properties id="db1" location="classpath:db.properties">
</util:properties>
```
