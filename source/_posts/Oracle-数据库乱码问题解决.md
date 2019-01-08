---
title: Oracle 数据库乱码问题解决
date: 2019-01-08 11:06:31
tags: 数据库
categories: 数据库
---

# 1 问题描述
下图中，**<font color=red>红色</font>** 的线表示乱码，**<font color=green>绿色</font>** 的线表示正常。  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/oracledatabase-env2.png)  

**3 种方式** 插入和查询数据 乱码情况如下：  
1、 数据库服务器 **本地** 用 **sqlplus** 登录，插入中文数据，查询结果 **<font color=red>乱码</font>** ！  
2、 Ubuntu **客户端** 用 **sqlplus** 登录，插入中文数据，查询结果  **<font color=red>乱码</font>** ！  
3、 Windows **客户端** 用 **PL/SQL** 登录，插入中文数据，查询结果 **<font color=green>正常</font>**。但是用前两种方式查询 此方式插入的中文数据，仍然是乱码。  

让我困惑的是，方式 3 正常，而 方式 1 乱码。  

# 2 问题排查

## 2.1 服务端 sqlplus 插入中文乱码
服务端 (CentOS) 的 字符集：  
```sh
locale
# LANG=zh_CN.UTF-8
```
zh 表示中文，CN 表示大陆地区， UTF-8 表示编码方式。  

数据库的 字符集：  
```sql
--Oracle数据库字符集 (主要)
select userenv('language') from dual;
--SIMPLIFIED CHINESE_CHINA.UTF8

--数据库字符集 (参考)
SELECT * FROM NLS_DATABASE_PARAMETERS;
--AMERICAN.UTF8

--实例字符集 (参考)
SELECT * FROM NLS_INSTANCE_PARAMETERS;
--AMERICAN
```
系统环境 和 数据库环境 都是 UTF-8 编码，为什么插入数据库的中文就乱码了呢？  

**<font color=red>原因</font>**：客户端 (通过localhost连接数据库，CentOS本身是服务端，也是客户端)  **NLS_LANG 环境变量** 没有设置 ！    

sqlplus 把 中文发给 Oracle 数据库时，会通过  **NLS_LANG 环境变量**  告诉数据库 (中文)字符串是以什么编码格式编码的，Oracle数据库有它自己的编码表(多个)，它会根据编码表对编码进行翻译和转码。最后，将转码之后的编码 存放到Oracle数据库中去。  

设置一下环境变量，乱码问题解决：  
```sh
export NLS_LANG='SIMPLIFIED CHINESE_CHINA.UTF8'
```
此时，通过 Windows PL/SQL 查询 CentOS 本地插入的 中文数据，也是正常的。  

## 2.2 远程客户端 sqlplus 插入中文乱码  
远程客户端 Ubuntu 系统的编码：  
```sh
locale
# LANG=zh_CN.UTF-8
```
同样的问题，Ubuntu 也没有 设置 **NLS_LANG 环境变量**。因此导致了乱码。  

设置  **NLS_LANG 环境变量** 字符集为 UTF-8 ：  
```sh
export NLS_LANG='SIMPLIFIED CHINESE_CHINA.UTF8'
```
远程客户端 sqlplus 插入中文乱码 问题解决。  
此时，通过 Windows PL/SQL 查询 Ubuntu 客户端插入的 中文数据，也是正常的。  

# 3 总结

数据库出现乱码时，应该逐一检查下列信息：   
1. 客户端操作系统字符集：**locale**      
2. 客户端操作系统 环境变量： **NLS_LANG**   
3. 服务端数据库字符集：**select userenv('language') from dual**    

应该将 三者 的字符集统一。  

# 参考  
 1、 [使用sqlplus执行sql时，有中文有乱码](https://blog.csdn.net/fyyinjing/article/details/77877239)   

 2、 [Oracle数据库中文乱码问题](https://www.cnblogs.com/xdouby/p/5666624.html)   

 3、 [Oracle 修改字符集---AL32UTF8 转换成UTF8字符集](https://blog.csdn.net/shiyu1157758655/article/details/78748283)  
