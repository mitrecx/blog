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

# 1. Spring 与 JDBC 整合  
Spring 提供了：
  * JdbcTemplate  
  * AOP事务管理  
  * 统一异常处理 DataAccessException  

0601 1:46
