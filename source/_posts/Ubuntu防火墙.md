---
title: Ubuntu防火墙
date: 2018-12-25 11:04:32
tags: Ubuntu
categories: Linux
---
# 1 防火墙简介
防火墙是位于 **内部网** 和 **外部网(一般指公网)** 之间的屏障，  
它按照系统管理员预先定义好的 **规则** 来控制数据包的进出。  
其作用是防止非法用户的进入。  

防火墙的特性：  
1、内部网络和外部网络之间的所有网络数据流都必须经过防火墙
2、只有符合安全策略的数据流才能通过防火墙  
3、应用层防火墙具备更细致的防护能力  

# 2 UFW

UFW (**<font color=red>Uncomplicated Firewall</font>**，简单的防火墙) is a program for managing a netfilter firewall designed to be easy to use.   
It uses a **command-line interface** consisting of a small number of simple commands(由少量简单命令组成的命令行接口), and uses **iptables** for configuration.     

## 2.1 ufw manul
NAME  
  ufw - program for managing a netfilter firewall  
DESCRIPTION  
  This program is for managing a Linux firewall and aims to provide an easy to use interface for the user.  
## 2.2 ufw 常用操作

1、   
**<font color=green>ufw [--dry-run] enable|disable|reload</font>**  打开|关闭|重启 防火墙  
**--dry-run**  试运行，显示结果并不执行  

2、   
**<font color=green>ufw [--dry-run] status [verbose|numbered]</font>** 查看状态  

3、   
ufw supports connection rate limiting, which is useful for **protecting  against  brute-force  login  attacks**.  
When a limit rule is used, ufw will normally allow the connection but will deny connections if an IP address attempts to initiate **6 or more connections within 30 seconds**.  
Typical usage is:
**<font color=green>ufw limit ssh/tcp</font>**  
如果同一个IP地址在30秒之内进行了6次及6次以上的连接，ufw将阻止 (deny) 该连接  

更多操作参考:  
```sh
man ufw
```
