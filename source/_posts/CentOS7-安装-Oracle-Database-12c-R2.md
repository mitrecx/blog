---
title: CentOS7 安装 Oracle Database 12c R2
date: 2019-01-03 23:39:48
tags: 数据库
categories: 数据库
---

# 1 了解 Oracle Database
[Oracle](https://www.oracle.com/database/) 数据库的版本 [[wiki](https://en.wikipedia.org/wiki/Oracle_Database#Releases_and_versions)]：  
1979年夏，Oracle第二版发布。  
1998年9月，**Oracle 8i**，该版本开始对 **Internet** 的支持。  
2001年6月，Oracle 9i。  
2003年9月，**Oracle 10g**，加入了 **网格(grid)计算** 的功能。  
2007年7月，Oracle 11g。  
2013年6月，Oracle发布了 **12c Release 1**，为 **云(cloud)计算** 设计。引入 **CDB**(Container Database,数据库容器)与 **PDB**(Pluggable Database,可插拔数据库)的新特性。  
**<font color=red>2016年9月</font>**，Oracle Database **12c Release 2**，本文安装的版本。  
2018年2月，Oracle Database 18c，Polymorphic(多态的) Table Functions, Active Directory Integration。  

# 2 安装 Oracle Database 12c Release 2

前言：  
在华为买了一个云主机，专门装 Oracle 数据库用的。  
买来时装的是 Ubuntu16.04，但是 Oracle Database 不直接支持 Ubuntu。  
于是换成 Oracle Linux 7，OL7安装数据库(Oracle Installation Prerequisites可以自动配置)方便，可是安装图形化界面时 yum 源改了几次都不行。  
于是换成 **RedHat 6**，**Fedora 26**，CoreOS 还是出错。  
最后 装了 n 次 **CentOS 7** ，在 CentOS 7 上装了 n 次 数据库， 建了 n 次 库，终于成功了。  

这一过程整整花了10天，难过(ಥ﹏ಥ)。。。。  
linux包管理工具一直都用apt，这次接触到了[rpm](https://www.cnblogs.com/LiuChunfu/p/8052890.html) ，yum，dnf。还有一些额外收获。   

下面进入正题。

----

## 2.0 CentOS 7 硬件配置情况
配置swap：  
```sh
dd if=/dev/zero of=/home/swap bs=1024 count=2097152 #bs为单位，count为设置的大小2048*1024
mkswap  /home/swap     #格式化交换文件
swapon  /home/swap     #立即启用交换分区文件, 要停止使用新创建的swap文件,只要执行 swapoff/home/swap命令即可.
```
为了避免 swap 分区在重启后失效，<code>vim /etc/fstab</code> 添加一行:  
```sh
/home/swap             swap          swap    defaults        0 0
```

查看内存分配情况:  
```sh
free -m

# 结果
            total        used        free      shared  buff/cache   available
Mem:         3789        1065         218         963        2504        1422
Swap:        2047           8        2039
```
配置 2G 的 swap 以后的机器：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-04_002235.png)  

## 2.1 下载
[Oracle Database 12c Release 2 (12.2.0.1.0)下载](https://www.oracle.com/technetwork/database/enterprise-edition/downloads/oracle12c-linux-12201-3608234.html)   
先下载到本地(Window10)，然后用 **<font color=red>Xshell 6</font>** ssh 连接 CentOS7 ，点击 “**新建文件传输(Xftp)**” 可以把下载好的文件上传到 CentOS7 上。  

上传完了，给 CentOS7 安装图形界面：  
安装X Window System  
```sh
yum groupinstall "X Window System"
```
安装图形桌面 GNOME(GNOME Desktop)  
```sh
yum groupinstall "GNOME Desktop"
```
Chrome 打开 华为云主机控制台 页面，点击 "远程登录" ，登录后验证图形桌面是否安装成功：  
```sh
init 5
```
如果出现以下图形桌面表示安装成功：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-04_090238.png)  

注：  
虽然 Oracle 数据库支持 silent 安装(没有图形桌面也能安装)，  
但考虑到 silent 安装 配置 rsp 文件有些繁琐，  
不如 先安装图形桌面，然后直接安装 Oracle数据库，比较方便。  

## 2.2 安装 Oracle 数据库

[安装方法只用三步](https://oracle-base.com/articles/12c/oracle-db-12cr2-installation-on-oracle-linux-6-and-7#Installation):  
1. 解压安装文件  
2. 配置安装必要条件  
3. 安装  

### 2.2.1 解压数据库安装文件  
```sh
unzip linuxx64_12201_database.zip
```
### 2.2.2 Oracle Installation Prerequisites
如果 Linux 版本是 Oracle Linux 的话，可以自动设置(Automatic Setup)，两行命令就把 **安装必要条件** 全部设置好了。  
但是我的 Linux 版本是 CentOS7，只能手动设置(Manual Setup)：  

0、 配置hostname (参考 如下两张图，一张是 CentOS7，一张是 redhat6.4)：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-17_154628.png)  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-17_153939.png)  

1、<code>vim /etc/sysctl.conf</code> 添加下列内容：  
```
fs.file-max = 6815744
kernel.sem = 250 32000 100 128
kernel.shmmni = 4096
kernel.shmall = 1073741824
kernel.shmmax = 4398046511104
kernel.panic_on_oops = 1
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.default.rp_filter = 2
fs.aio-max-nr = 1048576
net.ipv4.ip_local_port_range = 9000 65500
```
运行下列命令改变内核参数：  
```sh
/sbin/sysctl -p
```

2、 <code>vim /etc/security/limits.d/oracle-database-server-12cR2-preinstall.conf</code> 添加下列内容：  
```
oracle   soft   nofile    1024
oracle   hard   nofile    65536
oracle   soft   nproc    16384
oracle   hard   nproc    16384
oracle   soft   stack    10240
oracle   hard   stack    32768
oracle   hard   memlock    134217728
oracle   soft   memlock    134217728
```
上面的内容在 <code>/etc/security/limits.conf</code> 中也添加一遍。  

3、安装依赖包：  
```sh
yum install binutils -y
yum install compat-libcap1 -y
yum install compat-libstdc++-33 -y
yum install compat-libstdc++-33.i686 -y
yum install glibc -y
yum install glibc.i686 -y
yum install glibc-devel -y
yum install glibc-devel.i686 -y
yum install ksh -y
yum install libaio -y
yum install libaio.i686 -y
yum install libaio-devel -y
yum install libaio-devel.i686 -y
yum install libX11 -y
yum install libX11.i686 -y
yum install libXau -y
yum install libXau.i686 -y
yum install libXi -y
yum install libXi.i686 -y
yum install libXtst -y
yum install libXtst.i686 -y
yum install libgcc -y
yum install libgcc.i686 -y
yum install libstdc++ -y
yum install libstdc++.i686 -y
yum install libstdc++-devel -y
yum install libstdc++-devel.i686 -y
yum install libxcb -y
yum install libxcb.i686 -y
yum install make -y
yum install nfs-utils -y
yum install net-tools -y
yum install smartmontools -y
yum install sysstat -y
yum install unixODBC -y
yum install unixODBC-devel -y

yum install gcc -y
yum install gcc-c++ -y
yum install libXext -y
yum install libXext.i686 -y
yum install zlib-devel -y
yum install zlib-devel.i686 -y
```

4、 创建用户组合用户：  
```sh
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
groupadd -g 54324 backupdba
groupadd -g 54325 dgdba
groupadd -g 54326 kmdba
groupadd -g 54327 asmdba
groupadd -g 54328 asmoper
groupadd -g 54329 asmadmin
groupadd -g 54330 racdba
# 创建 oracle 用户
useradd -u 54321 -g oinstall -G dba,oper oracle

passwd oracle
```

5、 <code>vim /etc/selinux/config</code> 修改 SELINUX 标识：  
```sh
SELINUX=permissive
```
重启或执行以下命令 (root)：  
```sh
setenforce Permissive
```

6、 关闭防火墙 (root)：  
```sh
systemctl stop firewalld
systemctl disable firewalld
```

7、 创建安装目录：  
```sh
mkdir -p /u01/app/oracle/product/12.2.0.1/db_1
chown -R oracle:oinstall /u01
chmod -R 775 /u01
```

8、 创建环境变量文件 setEnv.sh :  
```sh
mkdir /home/oracle/scripts
```
```sh
cat > /home/oracle/scripts/setEnv.sh <<EOF
# Oracle Settings
export TMP=/tmp
export TMPDIR=\$TMP

export ORACLE_HOSTNAME=ol7-122.localdomain
export ORACLE_UNQNAME=cdb1
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=\$ORACLE_BASE/product/12.2.0.1/db_1
# export ORACLE_SID=cdb1
export ORACLE_SID=orcl

export PATH=/usr/sbin:/usr/local/bin:\$PATH
export PATH=\$ORACLE_HOME/bin:\$PATH

export LD_LIBRARY_PATH=\$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=\$ORACLE_HOME/jlib:\$ORACLE_HOME/rdbms/jlib
EOF
```
<code>/home/oracle/.bash_profile</code> 引用 setEnv.sh 文件：  
```sh
echo ". /home/oracle/scripts/setEnv.sh" >> /home/oracle/.bash_profile
```

9、 Create a "start_all.sh" and "stop_all.sh" script that can be called from a startup/shutdown service.  
```sh
cat > /home/oracle/scripts/start_all.sh <<EOF
#!/bin/bash
. /home/oracle/scripts/setEnv.sh

export ORAENV_ASK=NO
. oraenv
export ORAENV_ASK=YES

dbstart \$ORACLE_HOME
EOF


cat > /home/oracle/scripts/stop_all.sh <<EOF
#!/bin/bash
. /home/oracle/scripts/setEnv.sh

export ORAENV_ASK=NO
. oraenv
export ORAENV_ASK=YES

dbshut \$ORACLE_HOME
EOF

chown -R oracle.oinstall /home/oracle/scripts
chmod u+x /home/oracle/scripts/*.sh
```

### 2.2.3 安装

打开 Chrome 浏览器，打开 华为云主机控制台，登录 oracle 用户，打开图形桌面：  
```sh
init 5
```

进入 解压后的 database 目录，执行：   
```sh
./runInstaller
```

接下来会出现一个 Oracle 安装窗口，**<font color=red>建议每次点击next之前，阅读清楚选项的意义。</font>**    

安装完成后，检查一下环境变量 **<font color=red>$ORACLE_SID</font>**：  
```sh
#多数环境变量在 /home/oracle/scripts/setEnv.sh 文件中设置
env | grep -i oracle

# /etc/oratab中的 sid 一定要和 $ORACLE_SID 一致
vim /etc/oratab
```


# 3 建库
终端执行(需要图形桌面环境)：  
```sh
dbca
```
弹出 **Database Configuration Assistant** 窗口，默认next 创建数据库。  

# 4 登录

## 4.1 登录配置

**listener.ora 和 tnsnames.ora 配置 的 实例名(或服务名) 应该和 $ORACLE_SID 一致**。  

配置 listener.ora  [ [listener.ora文件介绍](https://docs.oracle.com/cd/B28359_01/network.111/b28317/listener.htm#NETRF008) ]:   
```sh
vim /u01/app/oracle/product/12.2.0.1/db_1/network/admin/listener.ora

#或者
vim $ORACLE_HOME/network/admin/listener.ora
```
内容如下：   
```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = mitre)(PORT = 1521))
    )
  )

ADR_BASE_LISTENER = /u01/app/oracle

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl)
        (SID_NAME = orcl)
    )
  )
```
配置 tnsnames.ora  [ [tnsnames.ora文件介绍](https://docs.oracle.com/cd/B28359_01/network.111/b28317/tnsnames.htm#NETRF007) ]:  
```sh
vim $ORACLE_HOME/network/admin/tnsnames.ora
```
内容如下：  
```
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = mitre)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = orcl)
    )
  )
```


查看 数据库的 **监听器** 状态：  
```sh
lsnrctl status

lsnrctl start #打开监听器
lsnrctl stop #关闭
```


## 4.2 启动数据库 与 sqlplus 登录方法
启动数据库：  
```sh
# 打开 sqlplus
sqlplus / as sysdba
## 打开成功会 弹出 SQL> 提示符

# 启动数据库
startup

# 用户登录
# conn 用户名/密码
conn system/123
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-04_153831.png)  

在数据库启动的情况下，可以直接在 shell 终端进行 用户登录 ：  
<code> sqlplus 用户名/密码@ip:port/SID </code>  
```sh
sqlplus system/123@localhost:1521/orcl
```

## 4.3 利用 PL/SQL 远程登录
Chrome 打开 华为云主机控制台，添加 **安全组-入规则** ，添加 1521 端口。  
安装 Oracle client 和 PL/SQL。  
登录结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-04_102804.png)  

# 参考
1、 [阿里云 CentOS7.4，静默安装Oracle11g2的教程](https://blog.csdn.net/sinat_32998977/article/details/79437014)   

2、 [CentOS7安装图形界面](https://blog.csdn.net/wang465745776/article/details/80161153)  

3、 [Oracle Database 12c Release 2 (12.2) Installation On Oracle Linux 7](https://oracle-base.com/articles/12c/oracle-db-12cr2-installation-on-oracle-linux-6-and-7#Installation)  
