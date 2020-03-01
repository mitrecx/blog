---
title: postgreSQL简单配置
date: 2020-02-28 13:17:51
tags: 数据库
---
# 卸载 yum 方式安装的 postgreSQL  

用以下两个命令查看 **系统信息** :  
```sh
lsb_release -a

uname -a
```

卸载 postgreSQL :  
```sh
# yum 删除软件包
sudo yum remove postgresql*

# 删除相关目录文件：
sudo rm -rf  /var/lib/pgsql
sudo rm -rf  /usr/pgsql*

# 删除pg相关用户组/用户
sudo userdel -r postgres
sudo groupdel postgres
```

# 源码方式安装 postgreSQL
[postgreSQL 官网](https://www.postgresql.org/download/) 介绍的安装方式有多种.  

这里简单介绍一下 编译源码 这种安装方式:  
1, 下载源码[(点击下载)](https://www.postgresql.org/ftp/source/);   
2, 解压并进入 postgresql-version 目录, 打开 INSTALL
3, 按照 INSTALL 文件的 指示安装:  
```sh
./configure
make
su
make install
adduser postgres
mkdir /usr/local/pgsql/data
chown postgres /usr/local/pgsql/data
su - postgres
# 初始化数据库
/usr/local/pgsql/bin/initdb -D /usr/local/pgsql/data
# 启动
/usr/local/pgsql/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start

# 创建数据库 test
/usr/local/pgsql/bin/createdb test
# 以 postgres role 连接到 test 数据库
/usr/local/pgsql/bin/psql test
```

提示:  
如果 <code>./configure</code> 提示 缺少 lib 则安装对应的 lib , 比如:  
```sh
sudo yum install readline-devel -y
sudo yum install zlib-devel.x86_64 -y
```

start 与 stop :
```sh
# 切到 postgres 用户, 进入 ~/.bashrc , 把 bin 添加到 PATH
export PATH=$PATH:/usr/local/pgsql/bin

# start
pg_ctl -D /usr/local/pgsql/data/ -l logfile start
# status
pg_ctl -D /usr/local/pgsql/data/ -l logfile status
# stop
pg_ctl -D /usr/local/pgsql/data/ -l logfile stop
```

# 远程访问 postgreSQL  

Client authentication is controlled by a configuration file,   
which traditionally is named **pg_hba.conf** and is stored in the database cluster's data directory.   
(HBA stands for **host-based authentication**.)   
A default pg_hba.conf file is installed when the data directory is initialized by initdb.   

修改 以下两个文件 :  
```sh
# 1, host-based authentication 配置文件:
sudo vim /usr/local/pgsql/data/pg_hba.conf
# 添加一行
host    all             all             0.0.0.0/0               md5

# 2, 全局配置文件:
sudo vim /usr/local/pgsql/data/postgresql.conf
# 修改一行
listen_addresses = '*'
# defaults to 'localhost'; use '*' for all
```

然后重启 postgreSQL .  

# 修改 role 密码  

terminal 截取片段:  
```
[postgres@shumingly ~]$
[postgres@shumingly ~]$ psql -h localhost -U postgres test
psql (12.0)
Type "help" for help.

test=# ALTER USER postgres WITH PASSWORD '国际通用密码';
ALTER ROLE
test=# \q
[postgres@shumingly ~]$
```
