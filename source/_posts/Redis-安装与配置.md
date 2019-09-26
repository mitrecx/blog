---
title: Redis 安装与配置
date: 2019-09-26 14:14:56
tags: Redis
categories: Redis
---

# 1 Redis 简介

[Redis](https://redis.io/topics/introduction) is an open source (BSD licensed), **in-memory data structure store**, used as a database, cache and message broker.  
redis是一个开源 (bsd许可) 的 **内存数据结构存储**，用作数据库、缓存和消息代理。  

Redis is written in ANSI C and works in most POSIX systems like Linux, \*BSD, OS X without external dependencies.  
Redis 用 SNSI C 语言编写, 能够无需任何外部依赖地 运行在大多数 POSIX 系统中, 如, Linux, BSD, OS X.  

目前, [Redis 官网](https://redis.io) 提供的最新版本是 5.0.5.  

# 2 阿里云主机 CentOS 安装并配置 Redis

## 2.1 [下载与安装](https://redis.io/download):  
```bash
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
tar zxf redis-5.0.5.tar.gz
cd redis-5.0.5
make
```

## 2.2 配置
在 解压后的目录 redis-5.0.5 下, 有一个 redis.conf 文件.  
redis.conf 就是 Redis 服务器的配置文件.  

修改 redis.conf 文件, 设置外网可访问:  
```sh
# 关闭保护模式
protected-mode no
# 不限 IP 访问
bind 0.0.0.0
# 修改默认端口
port 6342
# 设置密码 为123
requirepass 123
```

把 redis-5.0.5/src 添加到 PATH 环境变量:  
```sh
# 新建一个 sh 脚本
vim /etc/profile.d/redis.sh
# 在脚本中 添加环境变量
export PATH=$PATH:绝对路径/redis-5.0.5/src
# 使配置立即生效
source /etc/bashrc
```

## 2.3 配置 云主机的 安全组
如下开放 6342 端口:  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog2019/pic09/redis.png?Expires=1569483171&OSSAccessKeyId=TMP.hVdAfGhbtviU16RhyBhYqDm1fd7RgtBPFxVNSq4PixKWMckSgzUFnAdR6teGokySgJsS1SpnoCNgbiAbUbDAVrAPtksaX3LTEkf6AvPXyw8WtvSVXK3BZkwdnkNk2r.tmp&Signature=rg%2Fa11ojkZgh%2F9erPwzM7%2Bhj0FU%3D)

## 2.4 运行 Redis
```sh
# 运行 Redis
redis-server redis.conf

# 连接
redis-cli -h IP地址 -p 端口号 -a 密码
```

# 3 Redis 数据类型
支持五种 基本的 数据类型：  
1, string (字符串)   
2, hash (哈希)  
3, list (列表)   
4, set (集合)  
5, zset (sorted set: 有序集合)  

## 3.1 string
**string 类型** 是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB.  
```sh
127.0.0.1:6342> SET name "Mitre"
OK
127.0.0.1:6342> GET name
"Mitre"
```

## 3.2 hash
**hash 类型** 是一个 key(string 类型) 和 value 的映射表，hash 特别适合用于存储对象.  
```sh
127.0.0.1:6342> HMSET person name "Mitre" age 24
OK
127.0.0.1:6342> HGET person name
"Mitre"
127.0.0.1:6342> HGET person age
"24"
```

## 3.3 list 双向链表  
**list 类型** 是简单的字符串列表，按照插入顺序排序。  
可以添加一个元素到列表的头部(左边)或者尾部 (右边).  
```sh
127.0.0.1:6342> LPUSH langs "Java" "C++" "Python"
(integer) 3
127.0.0.1:6342> LPUSH langs "C#"
(integer) 4
127.0.0.1:6342> RPUSH langs "C"
(integer) 5
127.0.0.1:6342> LRANGE langs 0 10
1) "C#"
2) "Python"
3) "C++"
4) "Java"
5) "C"
```

## 3.4 set
**set 类型** 是 string 类型的无序集合.  
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1).  
```sh
127.0.0.1:6342> SADD companies "IBM" "google"
(integer) 2
127.0.0.1:6342> SADD companies "amazon"
(integer) 1
127.0.0.1:6342> SMEMBERS companies
1) "google"
2) "IBM"
3) "amazon"

# set 插入重复数据时, 会忽略掉本次插入
127.0.0.1:6342> SADD companies "amazon"
(integer) 0
```

## 3.5 zset
zset 和 set 一样也是 string类型 元素的集合, 且不允许重复的成员.  
不同的是 每个元素 都会关联一个 double类型 的分数.  
redis正是通过分数来为集合中的成员进行从小到大的排序.  
zset的成员是唯一的, 但分数 (score) 可以重复.
```sh
127.0.0.1:6342> ZADD billboard 3 "Mitre" 6 "Cherry"
(integer) 2
127.0.0.1:6342> ZADD billboard 1 "Owen"
(integer) 1
127.0.0.1:6342> ZRANGEBYSCORE billboard 0 10
1) "Owen"
2) "Mitre"
3) "Cherry"
```
