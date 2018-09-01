---
title: Linux 用户管理
date: 2018-09-01 21:37:15
tags:
- Linux
- Ubuntu
categories:
- Linux
- Ubuntu
---

# Linux用户管理
## 1. 添加用户
```sh
#添加用户
useradd -s /bin/bash -m mitre
#设置密码
passwd mitre
```
-s: name of user's login shell  
-m: home directory  

## 2. 删除用户
```sh
userdel -r mitre
```
-r 表示:  
Files in the user's home directory will be removed along with the home directory itself and the user's mail spool.  
Files located in other file systems will have to be searched for and deleted manually.  
用户home目录中的文件将随home目录本身和用户的邮件卷一起删除。  
位于其他文件系统中的文件将必须手动搜索和删除。  

## 3. sudo
sudo  **/ˈsuːdoʊ/** :  
以前：superuser do  
现在：substitute user do  

如果用户XXX没有 **superuser 权限**，使用 sudo 时，会提示：  
<font color=red>XXX is not in the sudoers file. This incident will be reported.</font>

把XXX加入 sudoers 中：  
```sh
vim /etc/sudoers
```
添加:  **username &nbsp;&nbsp; ALL=(ALL:ALL) ALL **
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/sudoers.jpg)  

## 4. 用户的其他操作

```sh
#查看哪些用户在线
who -H

#切换用户 substitute
su username

#切换到root
sudo -s
```

The Unix command [su](https://en.wikipedia.org/wiki/Su_(Unix)), which stands for(代表，表示) substitute user is used by a computer user to execute commands with the privileges of another user account.


## 5. references
1. [sudo](https://en.wikipedia.org/wiki/Sudo)  
2. [sudo and sudoers](https://www.youtube.com/watch?v=YSSIm0g00m4)  
