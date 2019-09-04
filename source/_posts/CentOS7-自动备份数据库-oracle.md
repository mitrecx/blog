---
title: CentOS7 定时备份数据库-oracle
date: 2019-06-27 17:17:35
tags: 数据库
categories: 数据库
---

# 1 命令行 导入/导出 数据库表

导入/导出 oracle 数据库的表里的数据, 需要用到 oracle 数据库的两个命令: <code>imp</code> 和 <code>exp</code>.  
[安装](https://mitrecx.github.io/2019/01/03/CentOS7-%E5%AE%89%E8%A3%85-Oracle-Database-12c-R2/)过数据库的机器, 就会有这两个命令.  

## 1.1 exp 导出数据  

<code>exp</code> 导出 faext用户下的 所有的表(包括表结构和数据), 存储过程, 视图等:  
```sh
exp faext/password@47.96.111.222:1521/S460 file=/home/oracle/Documents/test.dmp owner=faext
```
导出参数解释 :
<code>exp</code> **<font color='#2eb82e'>用户名/密码@数据库ip:端口/SID</font> <font color='#ff9900'>file=导出文件名称</font> <font color='#1a75ff'>owner=数据用户名</font>**    

仅仅导出某几张表 :  
```sh
exp faext/password@47.96.111.222:1521/S460 file=/home/oracle/Documents/test.dmp tables=tableName1, tableName2
```

exp 命令 有许多参数可以选择, 参考 :  
```sh
exp help=y
```

## 1.2 imp 导入数据  

把 20190627.dmp 文件中的数据 全部(full=y)导入 指定数据库中:  
```sh
imp system/oracle@localhost:1521/S460 file=/home/oracle/Documents/20190627.dmp full=y
```

把 20190627.dmp 文件中的 指定表 tableName1 的数据 导入指定数据库中:  
```sh
imp system/oracle@localhost:1521/S460 file=/home/oracle/Documents/20190627.dmp tables=tableName1
```

imp 命令 有许多参数可以选择, 参考:  
```sh
imp help=y
```

# 2 Linux 定时任务  

CentOS 定时任务 是 通过 **cron(crond) 系统服务** 来控制的.  这个系统服务是默认是 开机启动的.  
查看 **cron(crond) 系统服务** :  
```sh
service crond status
```

## 2.1 定时任务示例  

配置 **crontab** 实现 定时执行 指定的脚本:  
```sh
# 打开配置文件:  
crontab -e

#把 以下内容 复制到 配置文件中
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=oracle
*/1 * * * * date > /home/oracle/Documents/1.txt
```

这样就完成了一个简单的 **定时任务**:   
每分钟执行一次 <code>date</code> 命令, 把执行结果 写入1.txt 文件.   

查看定时任务配置:  
```sh
crontab -l
```

查看定时任务 执行 **日志**:  
```sh
sudo tail -f /var/log/cron
```

回看一下 定时任务的 配置文件:  
```
# SHELL环境变量
SHELL=/bin/bash
# PATH环境变量
PATH=/sbin:/bin:/usr/sbin:/usr/bin
#邮件发送给指定的用户
MAILTO=oracle

# 每分钟执行一次
*/1 * * * * date > /home/oracle/Documents/1.txt
```

crontab 配置中 有一个坑:   
**所有的 用户的环境变量 都会失效** !   
**系统自带的环境变量会失效, 自己配置的环境变量也会失效.**    

从上面的内容可以看到 配置了 **PATH** 环境变量, 这样我们才能 各个 bin 中的命令.  

**<font color='#8a00e6'>crontab 文件定时代码格式</font>**:  
```
minute hour day month week command
```
**<font color='#1a75ff'>minute</font>**: 表示分钟，可以是从0到59之间的任何整数。  
**<font color='#1a75ff'>hour</font>**: 表示小时，可以是从0到23之间的任何整数。  
**<font color='#1a75ff'>day</font>**: 表示日期，可以是从1到31之间的任何整数。  
**<font color='#1a75ff'>month</font>**: 表示月份，可以是从1到12之间的任何整数。  
**<font color='#1a75ff'>week</font>**: 表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。  
**<font color='#1a75ff'>command</font>**: 要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。  

星号 (\*)：代表所有可能的值，例如: month字段是星号 表示 (在满足其它字段的制约条件后) 每月都执行该任务.  
逗号 (,)：用逗号隔开的值指定一个列表范围，例如: hour段 "1,4,17" 表示 1点4点17点执行该任务.  
正斜线 (/)：正斜线指定时间的间隔频率，例如: minute段 "\*/2" 表示每(间隔)两分钟执行一次.  

## 2.2 定时备份数据库表  
shell 脚本如下, (每天)备份 faext 用户的所有数据,  只保留 10天 之内的备份文件:  
```sh
#!/bin/bash
. /etc/profile
# set environment variable
. /home/oracle/.bash_profile

bakPath='/home/oracle/Documents/faextDBBak';

#
currentDate=`date +%Y%m%d`;

# remove data 10 days ago
find ${bakPath} -mtime +10 -name "*.dmp" -exec rm -rf {} \;

# back up remote database tables
/u01/app/oracle/product/12.2.0.1/db_1/bin/exp faext/passwd@47.96.111.222:1521/S460 file=${bakPath}/${currentDate}.dmp owner=faext &> /dev/null;

# chmod
chmod 755 ${bakPath}/${currentDate}.dmp;
```

crontab 文件(<code>crontab -e</code>)配置 如下, 每天 19:00 执行定时任务:    
```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=oracle

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
0 19 * * * sh /home/oracle/Documents/faextDBBak/DBBak.sh
```

## 注意:  
一: **<font color=red>crontab 默认是没有 环境变量的</font>**, 一定要自己配置, 否则定时任务不会成功执行.  
二: /etc/crontab 是 root 级的定时任务配置文件, 普通用户 用<code>crontab -e</code> 配置.  
