---
title: MySQL 安装与命令行操作
date: 2018-09-01 21:43:12
tags:
- MySQL
- 数据库
categories: 数据库
---
# MySQL

## 1. Ubuntu 安装 MySQL
 ```sh
 #安装 mysql-server 会要求输入 root密码
 sudo apt-get install mysql-server  
 apt-get isntall mysql-client
 sudo apt-get install libmysqlclient-dev
 ```
查看是否安装成功：
 ```sh
 sudo netstat -apt | grep mysql
 ```
如果看到有mysql 的socket处于 listen 状态则表示安装成功。  

## 2. MySQL命令行

### 2.1 登录MySQL

```sh
mysql -u root -p
# -u root 表示用 root 账户登录
# -p 表示 password
```
本地登录 MySQL 没有问题，但是远程登录的时候，默认是拒绝连接的，这是配置文件默认配置的问题。  
```sh
cd /etc/mysql
vim mysql.conf.d/mysql.cnf
# 修改 系统变量 bind-address 为0.0.0.0 表示绑定地址任意
# bind-address            = 0.0.0.0

service mysql restart
```  
更多 [server系统变量](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html) 详细配置 参见官方文档。
## 2.2 查看数据库

### 2.2.1显示数据库
```MySQL
show databases
```
如果已经用 root 用户登录了，如下图，可以看到默认的 4 个数据库(不包括DBmitre，这是我后来创建的)。  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-8-23/showDatabases.png)  

### 2.2.2连接数据库： use <数据库名>
```MySQL
 --查看当前使用的数据库，默认是NULL
 select database();

 --选择数据库 DBmitre
 use DBmitre;
```

## 2.3 创建和删除数据库

```MySQL
--create database <数据库名>
create database DBmitre;

--drop database <数据库名>
drop database DBmitre;
```

## 2.4 创建用户
1、增加一个用户 user1 密码为 password1，让他可以在任何主机上登录( **主机地址填写为%** )，并对所有数据库有查询、插入、修改、删除的权限。首先用root用户连入MYSQL，然后键入以下命令：  
```MySQL
grant select,insert,update,delete on *.* to use1r@% Identified by “password1”;
```

2、增加一个用户 user2 密码为 password2,让他只可以在localhost上登录，并可以对数据库 DBmitre 进行查询、插入、修改、删除的操作（localhost指本地主机，即MYSQL数据库所在的那台主机），这样用户即使用知道user2的密码，他也无法从internet上直接访问数据库，只能通过MYSQL主机上的web页来访问了。  
```MySQL
grant select,insert,update,delete on DBmitre.* to user2@localhost identified by “password2”;
```
3、如果你不想 user2 有密码，可以再打一个命令将密码消掉。   
```MySQL
grant select,insert,update,delete on DBmitre.* to user2@localhost identified by “”;
```
## 2.5 其他命令
 ```mysql
 --显示MYSQL的版本
 select version();

--当前选择的数据库
select database();

--当前用户及数据库连接地址
select user();
 ```
# 3. 字符集问题
查看表的字符集  
```MySQL
show create table table_name \G;

--查看字符集
show variables like '%char%';
```

修改 **表** 的字符集：  
```MySQL
ALTER TABLE table_name DEFAULT CHARACTER SET character_name;
```
同时修改 **表和列** 的字符集：  
```MySQL
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8;
```


注：也可以修mysql配置文件/etc/my.cnf 来改变字符集。  
在 **[mysqld]** 下 添加 **character_set_server=utf8**  

**修改字符集，然后重启。**

**<font color=red>JDBC 连接MySQL 这样还不能完全解决乱码问题，因为 JDBC 驱动默认的字符集 是 ISO-8859-1</font>**。  
解决方法：在连接URL中加  **“?useUnicode=true&characterEncoding=utf8”** 来指定 JDBC 驱动连接数据库的字符集。   

```java
public static Connection getConnection() throws Exception {
		String driver="com.mysql.jdbc.Driver";
		String url="jdbc:mysql://cherry.mitrecx.cn:3306/DBmitre";
		Connection conn=null;
		try {
			Class.forName(driver);
			conn=DriverManager.getConnection(url+"?useUnicode=true&characterEncoding=utf8","root","1234567");
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
		return conn;
	}
```
注：  
Oracle 数据库不会存在这样字符编码的问题。


# references
1. [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
2. [Mysql常用命令行大全](https://www.cnblogs.com/bluealine/p/7832219.html)
