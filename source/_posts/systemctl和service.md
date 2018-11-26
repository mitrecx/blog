---
title: systemctl和service
date: 2018-11-23 15:59:01
tags: ["Linux","Ubuntu"]
categories: Linux
---

内核启动第一个用户空间进程是由 **init** (/sbin/init)开始的。它主要负责 启动和终止系统中的基础服务进程。  

linux系统中主要的 **init版本** 有如下:  
**<font color=red>System V init</font>**:	传统的init,大多数的linux发行版都会兼容。  
**<font color=green>Systemd</font>**:	新的init,很多linux发行版都已经或者计划转向Systemd。    

查看 1 号进程：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-26_150028.png)  
/sbin/init 是 /lib/systemd/systemd 的链接文件。**系统的 1 号进程的确是 systemd**，只不过在 ubuntu 系统中被起了个别名叫 /sbin/init。   
注：0号进程->1号内核进程->1号用户进程（init进程）->getty进程->shell进程。  

# 1 service 命令  
**service - run a <font color=red>System V init</font> script**  

service是一个命令，服务配置文件存放目录/etc/init.d/。   
service是去 /etc/init.d 目录下执行相关脚本程序。   

**service 的用法**  

service 操作：  
```shell
service SCRIPT COMMAND [OPTIONS]
```
SCRIPT 即是 **/etc/init.d** 目录下的文件。  
COMMAND 包含：**start, stop, restart, reload, status.**  
例如：  
```sh
#通过ssh.service，输出服务类型 unit 的基本信息
service ssh status
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-26_143027.png)  
1、第一行是对 unit 的描述。  

2、Loaded 描述操作系统启动时会不会启动这个服务：  
**enabled**： 表示开机时启动。  
**disabled**： 表示开机时不启动。    
**static**：这个 unit 不可以自己启动，不过可能会被其它的 enabled 的服务来唤醒。
**mask**：这个 unit 无论如何都无法被启动！因为已经被强制注销。可通过 systemctl unmask 改回原来的状态。

3、 active 的状态：  
**active (running)**： 表示服务正在运行中。  
**inactive (dead)**： 表示服务当前没有运行。  
**active (exited)**：仅执行一次就正常结束的服务，目前并没有任何程序在系统中执行。  
举例来说，开机或者是挂载时才会进行一次的 quotaon 功能，就是这种模式。   
Quotaon 不需要一直执行，只在执行一次之后，就交给文件系统去自行处理。  
通常用 bash shell 写的小型服务，大多是属于这种类型。  
**active (waiting)**：正在执行当中，不过还再等待其他的事件才能继续处理。举例来说，打印的相关服务就是这种状态。

service 查看所有 **服务** 状态：  
```shell
service --status-all
```
其中 "+" 代表服务正在运行， "-" 代表服务处于关闭状态，"?" 代表根本没有状态。  

# 2 systemctl 命令
**systemctl - Control the <font color=green>systemd</font> system and service manager**  

**ubuntu自从15.04版本以后都使用了systemd。**

systemctl 用法：   
```sh
systemctl [OPTIONS...] COMMAND [NAME...]
```
COMMAND 包含：start, stop, restart, reload, status.
NAME 是 服务名。  

例：  
```sh
# 查看 ssh服务的状态
systemctl status ssh

#打开服务
sudo systemctl start ssh
#关闭服务
sudo systemctl stop ssh
#重启服务
sudo systemctl restart ssh
#不中断正常功能下重新加载服务
sudo systemctl reload ssh
#设置服务的开机自启动
sudo systemctl enable ssh
#关闭服务的开机自启动
sudo systemctl disable ssh

#查看活跃的单元
systemctl list-units
```

查看默认的 target：  
```sh
systemctl get-default  
```
systemd 也提供了几个简单的指令用来切换操作模式:  
```sh
systemctl poweroff  # 系统关机            
systemctl reboot    # 重新开机            
systemctl suspend   # 进入暂停模式        
systemctl hibernate  # 进入休眠模式
```
**suspend**：暂停模式会将系统的状态保存到内存中，然后关闭掉大部分的系统硬件，当然，并没有实际关机。当用户按下唤醒机器的按钮，系统数据会从内存中回复，然后重新驱动被大部分关闭的硬件，所以唤醒系统的速度比较快。  
**hibernate**：休眠模式则是将系统状态保存到硬盘当中，保存完毕后，将计算机关机。当用户尝试唤醒系统时，系统会开始正常运行，然后将保存在硬盘中的系统状态恢复回来。因为数据需要从硬盘读取，因此唤醒的速度比较慢

检查 unit 之间的依赖性:  
```sh
systemctl list-dependencies [unit] [--reverse]  
#--reverse 会反向追踪是谁在使用这个 unit
```
例如：  
```sh
systemctl list-dependencies default.target
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-26_145027.png)  
当前运行在 graphical.target 下，它由一个长长的依赖列表(上图并未展示所有的项目)，其中最重要的依赖项目为 multi-user.target。可以使用 --reverse 选项查看 multi-user.target unit 被谁使用。

总结:   
systemctl 提供了管理 systemd 和系统服务的众多命令。  
systemd 是某些linux发行版用来代替系统启动 init 和 service 的一个新的系统工具，同时兼容init。  
