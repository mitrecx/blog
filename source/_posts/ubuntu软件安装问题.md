---
title: ubuntu软件安装问题
date: 2018-08-04 12:16:53
tags:
categories: Linux
---
今天想安装一个ksh，结果报错了。  
然后我发现，__install__ 任何软件都会报同样的错。  
错误如下：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/sourceList.png)  
这无疑是 __软件源__ (<font color=orange>/etc/apt/sources.list</font>)出问题了。  

解决办法：  
```sh
sudo rm /var/lib/apt/lists/* -vf

sudo apt-get update
```
