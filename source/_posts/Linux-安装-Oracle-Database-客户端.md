---
title: Linux 安装 Oracle Database 客户端
date: 2019-01-07 10:02:13
tags: 数据库
categories: 数据库
---
Oracle DataBase Client 版本和远程数据库的版本都是 12.2.0.1.0。  

# 1 下载 Instant Client Package (Basic 和 SQL*Plus)

[下载地址](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)  

下载两个压缩包：  
* instantclient-basic-linux.x64-12.2.0.1.0.zip   
* instantclient-sqlplus-linux.x64-12.2.0.1.0.zip  

# 2 解压

```sh
unzip instantclient-basic-linux.x64-12.2.0.1.0.zip
unzip instantclient-sqlplus-linux.x64-12.2.0.1.0.zip
```

```sh
sudo mkdir /opt/instantclient
mv instantclient_12_2/* /opt/instantclient
```

# 3 设置环境变量
把以下环境变量加入 <code>~/.bashrc</code>文件中 (或者 在 <code>/etc/profile.d</code> 目录下新建一个文件)：
```sh
export ORACLE_HOME=/opt/instantclient
export LD_LIBRARY_PATH=/opt/instantclient
export TNS_ADMIN=/opt/instantclient
export PATH=$PATH:/opt/instantclient
```

# 4 连接 Oracle 数据库
```sh
sqlplus system/123@114.115.165.64:1521/orcl
```
