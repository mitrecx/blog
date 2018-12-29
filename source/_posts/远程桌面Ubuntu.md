---
title: 远程桌面Ubuntu
date: 2018-09-01 21:20:54
tags:
- 远程桌面
- xrdp
- Ubuntu
categories: 远程桌面
---
# 远程桌面Ubuntu
Linux 有 ssh ，Windows 有 Xshell ，那么还要远程桌面干嘛？？  
**ssh隧道 虽然强大**，但是在安装 Oracle Weblogic Server/Oracle数据库 的时候，ssh 将束手无策。  
有了图形桌面，在服务端安装 Oracle产品 将易如反掌。  


## 1. 服务端(Ubuntu)
不论是 Windows 还是 Linux 远程连接 Ubuntu 都需要在远程服务端安装 **桌面环境** 。  

### 1.1 xrdp
[xrdp](http://www.xrdp.org/): An open source **<font color=red>remote desktop protocol(rdp)</font>** server.   
xrdp uses **the remote desktop protocol** to present a **GUI(Graphical User Interface)** to the user.  

### 1.2 在Ubuntu上安装桌面环境  
```sh
# 安装xrdp
sudo apt-get install xrdp
```
仅仅安装 xrdp 协议，还不能在windows上使用 远程桌面 连接到Ubuntu。  
还要安装vncServer才行：  
```sh
# 安装vnc4server
sudo apt-get install vnc4server

# 安装xubuntu-desktop
sudo apt-get install xubuntu-desktop

# 向xsession中写入xfce4-session(每个用户需要单独运行)
echo "xfce4-session" >~/.xsession
```

开启xrdp服务
```sh
sudo service xrdp restart
```

## 2. 本地端(Linux/Windows)
### 2.1 本地为 Windows 环境
打开 Windows 自带的 **远程桌面连接** 工具，输入 **服务端** IP 和 用户名，密码 就可以连接了。  
### 2.2 本地为 Linux 环境
暂时没试，以后再说。  
2018-8-21
