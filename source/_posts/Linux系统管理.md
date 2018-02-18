---
title: Linux系统管理
date: 2018-02-18 09:31:41
tags: Linux
---

# root: 系统管理员的登录
管理员主要使用root用户ID履行职责。
```sh
grep "^root" /etc/passwd
```
以上命令输出可以看出root用户ID为0，事实上，任何一个以0为事实UID的进程都具有root的能力。  

```sh
su -
#有时候需要用sudo执行
sudo su -
```
“ - ” 参数表示使用root的登录环境； 如果想保留当前用户的环境（如，主目录），就不用参数 " - "。  
root的提示符（PS1）通常是 #  

su不仅可以切换到root用户，su可以切换到任何一个用户(su [opt] [uname])：  
```sh
su mitre
su mitrecx
```

# 用户管理
## 理解/etc/passwd
除密码以外，所有用户信息都存储在/etc/passwd中。  
这个文件曾经包含了密码，所以人们还一直在用passwd称呼它。现在密码加密存储在/etc/shadow。  
**login和passwd程序会查阅这两个文件，以完成用户身份验证。**  

passwd文件一共7个字段：  
1. 用户名

2. 密码：早期UNIX系统的密码是放在这个文件中的，但因为这个文件的特性是所有程序都能够读取，所以，这样很容易造成数据被窃取，因此后来就将这个字段的密码数据改放到/etc/shadow中了

3. UID：用户ID，每个账号名称对应一个UID，通常UID=0表示root管理员

4. GID：组ID，与/etc/group有关，/etc/group与/etc/passwd差不多，是用来规范用户组信息的

5. 用户信息说明栏： 用来解释这个账号是干什么的

6. 家目录：home目录，即用户登陆以后跳转到的目录，以root用户为例，/root是它的家目录，所以root用户登陆以后就跳转到/root目录这里

7. Shell：用户使用的shell，通常使用/bin/bash这个shell，这也就是为什么登陆Linux时默认的shell是bash的原因，就是在这里设置的，如果要想更改登陆后使用的shell，可以在这里修改。另外一个很重要的东西是有一个shell可以用来替代让账号无法登陆的命令，那就是/sbin/nologin。

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/Linux_etc_passwd.png)  

## 理解/etc/group
group一个4个字段：  
1. 组名  
2. 密码  
3. 组ID（GID）
4. 属于该组的用户列表  

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/Linux_etc_group.png)  
```sh
#添加用户组 gid为1314 gname为offer
groupadd -g 1314 offer
#详细参数man
```
## 修改和添加用户配置文件
```sh
useradd -u 210 -g dba -c "THE RDBMS" -d /home/oracle -s /bin/ksh -m oracle
```
**useradd   --- create a new user or update default new user information**  

**<font color=green>-u, --uid UID</font>**  
The numerical value of the user's ID. This value must be **unique**, **unless** the -o option is used. The value must be non-negative. The default is to use the smallest ID value greater than or equal to UID_MIN and greater than every
other user.  
**<font color=green>-g, --gid GROUP</font>**  
**The group name** or number of the user's initial login group. The group name must exist. A group number must refer to an already existing group.  
If not specified, the behavior of useradd will depend on the USERGROUPS_ENAB variable in /etc/login.defs. If this variable is set to yes (or -U/--user-group is specified on the command line), a group will be created for the user, with the same name as her loginname.  
**<font color=green>-c, --comment COMMENT</font>**  
Any text string. It is generally **a short description** of the login, and is currently used as the field for the user's full name.  
**<font color=green>-d, --home-dir HOME_DIR</font>**  
The new user will be created using HOME_DIR as the value for the user's login directory. The default is to append the LOGIN name to BASE_DIR and use that as the login directory name. **The directory HOME_DIR does not have to exist but will not be created if it is missing.**(如果没有HOME_DIE目录，这个-d选项会失效)  
**<font color=green>-s, --shell SHELL</font>**  
The name of **the user's login shell**. The default is to leave this field blank, which causes the system to select the default login shell specified by the SHELL variable in /etc/default/useradd, or an empty string by default.  
**<font color=green>-m, --create-home</font>**  
**Create the user's home directory if it does not exist**. The files and directories contained in the skeleton directory
(which can be defined with the -k option) will be copied to the home directory.  

一个例子：  
```sh
sudo useradd -s /bin/bash -m sm
sudo passwd sm
sudo userdel -r sm
```
**userdel --- delete a user account and related files**  

**<font color=green>-r, --remove</font>**  
Files in the user's home directory will be removed along with the home directory itself and the user's mail spool.
Files located in other file systems will have to be searched for and deleted manually.(如果没有-r选项，就不会删除用户文件,需要自己手动删除)  

# 维护安全
**因为计算机系统中的安全性最终与文件相关**，所以错误的文件权限很容易被恶意用户用于hack。  
维护文件安全的三种机制：  
* 受限制的shell（可以在passwd的第七字段设置）  
* Set-User-Id(SUID): 临时能力（不同的事实UID和真实UID）
* 粘着位

粘着位，在/tmp，/usr/tmp目录下，不同的用户创建的文件其他用户可以访问，但不可以删除。这就是通过粘着位实现的。  
```sh
chmod 1755 one.txt
ll one.txt
-rwxr-xr-t  1 one   one     54 2月  18 13:53 one.txt*
```
注意到，other用户的可执行的位置变成了t，即是 sticky bit。  
这样其他用户就不能删除不归它们所有的文件。（也不能写）  
粘着位的用处不大啊，不能删除的话，只要设置给other权限为" r - x "不就行了。。。  

# 启动与关机
计算机加电后，**系统** 会查找所有外围设备，然后执行一系列程序，**最终把内核加载到内存中**。  
内核随后生成init ( PID为1 )，init再进一步生成其他进程。  
**init通常做为 initialization(n,初始化) 的缩写使用**  
正常情况下，系统会处于以下这些系统相关的 **运行级别（run-level）** 之一：  
0 —— 系统关机  
1 —— 系统管理模式（加载了本地文件系统）  
2 —— 多用户模式（NFS不可用）  
3 —— 完全用户模式  
5 —— Linux **图形环境模式**  
6 —— 关机和重启模式  
s或S —— 单用户模式（加载文件系统）  

当系统重启时，init首先进入 **运行级别** 1或S，然后在转入用户模式（2、3或5）。  
当系统关闭时，init转为 **运行级别** 0或6 。  

管理员可以显式使用init命令，将系统转为任意 **运行级别** 。  
```sh
#查看运行级别
who -r
```

我在一天工作结束时，要关机了！  
使用shutdown命令关闭计算机，shutdown用wall通知所有用户：系统将要关闭，并给出注销指令。  
注：  
一些系统（如Solaris）提供了reboot和halt命令，他们也会关闭系统，但不会向用户发出警告。  
在管理多用户系统时，应当坚持使用shutdown。  

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/shutdown_k.png)  

# 文件系统
文件系统采用目录结构，有自己的根目录。  
操作系统通常驻留在多个文件系统上。在启动时，这些文件系统使用一种加载技术合并起来，向用户显示为单个文件系统。  
这些文件系统中有一个特殊系统—— **根文件系统**。它包含UNIX的基本要素——root目录、/bin、/etc、/dev、/lib目录。  
大多数系统还有 **交换文件系统** ，当系统内存中加载的内容过多时，内核会将一些进程移除内存，放到swap文件系统中。当这些交换的进程做好运行准备时，再将它们加载到内存。用户不能直接访问swap。  

除了这些基本的文件系统之外，还有更多的文件系统。**系统文件应当与用户创建的数据文件隔离存放**。因此，通常有一个/home文件系统来容纳所有用户，将/usr、/tmp等作为独立的文件系统。  

文件系统的4个组成部分：  
1. 启动块——这个块 **包含了** 一个小的 **启动程序和分区表**，它经常被称为主引导记录（MBR）。<font color=red>(原来，MBR在文件系统中)</font>  
2. 超级块——这个区域包含了有关文件系统的全局信息。包括iNode和数据块的空闲列表。  
3. inode块——包含了文件系统中 **每个文件的inode**。创建一个文件时，会在这里分配一个inode。  
4. 数据块——用户生成的 **所有数据和程序** 都存储在这一区域。  

**<font color=orange>文件系统类型</font>**:  
ext4,ext3  
nfs  
swap  
vfat  
fat32  
windows里还有NTFS  

# 加载和卸载文件系统
mount : mount a filesystem  
```sh
#mount device dir
mount  /dev/sda1 /tmp
```

umount: unmount file systems  
```sh
#umount {directory|device}
umount /dev/sda1
```
注：  
在使用mount -a时，在mount配置文件中列出的所有文件系统都将被加载。在系统启动时会执行mount -a 。  
同样道理，shutdown会运行 umount -a 。  

# fsck： 文件系统检查
**fsck（file system consistency check，文件系统一致性检查）**：  
update守护进程每30秒调用一次sync，将超级块和 inode 的内存副本写到磁盘。  
这一延迟为文件系统的不一致性创建了条件。如果在将超级块写到磁盘之前发生了掉电，文件系统将不在完整。  
**fsck 命令用于检查和修复被破坏的文件系统。**  

# 管理磁盘空间
## df: 查看磁盘空闲空间
**df (disk free，磁盘空闲)命令分别报告每个文件系统上的空闲空间数。**  
df - report file system disk space usage  

## du: 磁盘利用率
**我们经常要查看一个特定目录树的使用情况，而不是整个文件系统的耗用情况**。   
**du（disk usage，磁盘利用率）命令会递推查看目录树。**  
如果直接使用du，通常会显示很多文件（列出目录下的所有子目录，子目录的子目录...的使用情况）。一般我们可以使用 -s （summary）选项，只显示一层子目录，而且还是汇总的。  

查看/home目录下各用户耗用的空间量：    
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/du_sh.png)  

## fdisk
fdisk - manipulate disk partition table ，磁盘分区表操作工具；  
```sh
sudo fdisk -l
```
分别在不同主机上运行，结果如下：  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/0218_1.png)  
注：  
fdisk可以添加、删除、转换分区，自己还没有尝试，略略...  


# tar: 备份文件
**tar — The GNU version of the tar archiving utility**  

键选项（只能使用一个）：  

  |选项|意义|
  |--|--|
  |-c |创建一个新档案|
  |-x| 从档案中提取文件|
  |-t| 列出档案中的内容|
  |-r, -u| 在档案中追加文件, -u只追加比档案新的文件|

非键选项：

|选项|意义|
|--|--|
|-f| archive file，就是要操作的档案|
|-v|以长格式列出文件|
|-j| --bzip2  |
|-z| --gzip, --gunzip --ungzip|

测试：  
```sh
#把当前目录下的shell脚本备份到 666档案
mitre@mitre-TM1701666:~/Desktop$ tar -cvf 666 *.sh
hh.sh
ifStr.sh
set.sh
test.sh
tt.sh

#把 以f开头的文件 追加到 666
mitre@mitre-TM1701666:~/Desktop$ tar -uvf 666 f*
first_prog.c
foo

mitre@mitre-TM1701666:~/Desktop$ tar -tvf 666
-rwxrwxr-x mitre/mitre      56 2018-02-03 23:43 hh.sh
-rwxrwxr-x mitre/mitre     166 2018-02-03 15:28 ifStr.sh
-rwxrwxr-x mitre/mitre      77 2018-02-03 22:29 set.sh
-rwxrwxr-x mitre/mitre     209 2018-02-03 02:29 test.sh
-rwxrwxr-x mitre/mitre     268 2018-02-10 22:03 tt.sh
-rw-rw-r-- mitre/mitre     425 2018-02-07 13:32 first_prog.c
-rw-rw-r-- mitre/mitre     191 2018-01-31 18:01 foo

#备份目录
mitre@mitre-TM1701666:~/Desktop$ tar -jcvf dir.bzip2 Dir/
Dir/
Dir/rec_deposit
Dir/Makefile2
Dir/cdir/
Dir/cdir/kk
Dir/Makefile
Dir/rec_deposit.o
Dir/quit.o
Dir/quit.c
Dir/rec_deposit.c
Dir/arg_check.h
Dir/quit.h
Dir/arg_check.c
Dir/progs.tar
Dir/arg_check.o

```
