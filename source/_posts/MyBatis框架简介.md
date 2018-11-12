---
title: MyBatis框架简介
date: 2018-11-08 19:15:50
tags: java
categories: java
---

# 1 MyBatis 简介

什么是 [MyBatis](http://www.mybatis.org/mybatis-3/zh/index.html) ？
MyBatis 是一款优秀的 **持久层框架**，  
它支持定制化 SQL、存储过程以及高级映射。  
MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。  
MyBatis 可以使用简单的 XML 或注解来配置和 **映射** 原生信息，**将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录**。  

# 2 使用 MyBatis 开发流程
使用 MyBatis 框架操作数据库的步骤简图：  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/mybatis.png)  

上图中涉及到两个xml文件，一个是 **(全局)配置文件**，一个是 **映射文件**。  

**全局配置文件(configuration XML)** 内容，[参考 MyBatis 官网提供的模板](http://www.mybatis.org/mybatis-3/zh/getting-started.html)：  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

**映射文件(Mapper XML)** 示例，[参考 MyBatis 官网提供的模板](http://www.mybatis.org/mybatis-3/zh/getting-started.html)：  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

另外，**获取 SqlSession** 参考如下代码片段：  
```java
String resource = "sqlMapConfig.xml";

InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sSession = sqlSessionFactory.openSession();
```
**SqlSessionFactoryBuilder**   
这个类可以被实例化、使用和抛弃。一旦创建了 SqlSessionFactory，就不再需要它了。  

**SqlSessionFactory**  
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由对它进行清除和重建。  

**SqlSession**  
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的。所以它的最佳的作用域是请求或方法作用域。绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。  
如果你正在使用一种 web 框架，要考虑把 SqlSession 放在一个和 HTTP 请求对象相似的作用域中。  
换句话说，**每次收到一个 HTTP 请求，局应该打开一个 SqlSession，返回一个响应，就关闭 SqlSession**。  
下面的示例就是一个确保 SqlSession 关闭的 **标准模式**：  
```java
SqlSession session = sqlSessionFactory.openSession();
try {
  // do work
} finally {
  session.close();
}
```

**小结：**  
MyBatis 的真正强大在于它的映射语句。  
拿它跟具有相同功能的 JDBC 代码进行对比，省掉了 表字段和实体类属性之间赋值 的代码。  
MyBatis 就是针对 SQL 构建的，并且比普通的方法做的更好。  

# 3 MyBatis 增删改查示例  
根据数据库表的结构建立实体类 Emp.java ：  
```java
package cn.mitrecx.domain;
import java.math.BigDecimal;

public class Emp {
	//属性名 与 字段名 一致
	private Integer id;
	private String name;
	private Integer age;
	private BigDecimal salary;
  //get set方法略
}
```
映射文件(Mapper XML) EmpMapper.xml ：  
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
 "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="empSql">

	<select id="findEmp" resultType="cn.mitrecx.domain.Emp">
		select * from emp
	</select>

	<select id="findEmpByName" parameterType="java.lang.String" resultType="cn.mitrecx.domain.Emp">
		select * from emp
		where name like #{name}
	</select>

	<select id="findEmpById" parameterType="int" resultType="cn.mitrecx.domain.Emp">
		select * from emp
		where id=#{id}
	</select>

	<insert id="saveEmpByEmp" parameterType="cn.mitrecx.domain.Emp">
		insert into emp(name,age,salary)
		values(#{name},#{age},#{salary})
	</insert>
	<insert id="saveEmpByMap" parameterType="java.util.Map">
		insert into emp(name,age,salary)
		values(#{xm},#{nl},#{xs})
	</insert>

	<delete id="deleteEmpByName" parameterType="java.lang.String">
		delete from emp
		where name=#{name}
	</delete>

	<update id="updateEmpById" parameterType="java.util.Map" >
		update emp
		set salary=#{salary}
		where id=#{id}
	</update>
</mapper>
```
MyBatis全局配置文件 SqlMapConfig.xml ：  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

	<environments default="environment">
		<environment id="environment">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.cj.jdbc.Driver"/>
			 	<property name="url" value="jdbc:mysql://cherry.mitrecx.cn:3306/DBmitre?useUnicode=true&amp;characterEncoding=utf8"/>
			 	<property name="username" value="root"/>
			 	<property name="password" value="123456Seven"/>
			</dataSource>
		</environment>
	</environments>

	<!-- 加载SQL定义文件 -->
	<mappers>
		<mapper resource="cn/mitrecx/domain/EmpMapper.xml" ></mapper>
	</mappers>

</configuration>
```
测试类 ：  
```java
package cn.mitrecx.test;

import java.io.IOException;
import java.io.InputStream;
import java.math.BigDecimal;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import cn.mitrecx.domain.Emp;

public class TestEmp {
	public static void main(String[] args) throws IOException {
//		//加载主配置文件SqlMapConfig
//		Reader reader=Resources.getResourceAsReader("sqlMapConfig.xml");
//		SqlSessionFactory ssf=new SqlSessionFactoryBuilder().build(reader);
//		SqlSession sSession=ssf.openSession();
		String resource = "sqlMapConfig.xml";
		InputStream inputStream = Resources.getResourceAsStream(resource);
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		SqlSession sSession = sqlSessionFactory.openSession();
		System.out.println("MyBatis show");

		//List<Emp> list=sSession.selectList("findEmp");
		List<Emp> list=sSession.selectList("findEmpByName", "%e%");
		for(Emp e:list) {
			System.out.println(e.getName()+" "+e.getSalary());
		}

		System.out.println("-------单行查询测试---------");
		Emp e1=sSession.selectOne("findEmpById",10);
		System.out.println(e1.getId()+" "+e1.getName()+" "+e1.getSalary());

		System.out.println("-------插入测试---------");
		Emp e2=new Emp();
		e2.setName("coco");
		e2.setAge(11);
		e2.setSalary(new BigDecimal("8848"));
		sSession.insert("saveEmpByEmp", e2);
		sSession.commit();//自动提交默认是关闭的。增删改必须要提交
		Map<String,String> map=new HashMap<String,String>();
		map.put("xm", "kiki");
		map.put("nl", "23");
		map.put("xs", "12345");
		sSession.insert("saveEmpByMap",map);
		sSession.commit();

		System.out.println("-------删除测试---------");
		sSession.delete("deleteEmpByName", "coco");
		sSession.commit();
		System.out.println("-------修改测试---------");
		Map<String,String> map2=new HashMap<String,String>();
		map2.put("id", "10");
		map2.put("salary","111112");
		sSession.delete("updateEmpById", map2);
		sSession.commit();

		//释放session
		sSession.close();
	}
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-09_101010.png)  

# 4 动态 SQL  
* if  
* choose (when, otherwise)  
* trim (where, set)  
* foreach  

动态 SQL 通常要做的事情是根据条件包含 where 子句的一部分。比如：  
```xml
<select id="findActiveBlogWithTitleLike"
     resultType="Blog">
  SELECT * FROM BLOG
  WHERE state = ‘ACTIVE’
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```
[MyBatis官网对动态SQL做了详细的说明](http://www.mybatis.org/mybatis-3/zh/dynamic-sql.html)，这里不再展开。  
