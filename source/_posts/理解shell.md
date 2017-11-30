---
title: 理解shell
date: 2017-11-30 18:06:57
tags:
  - tech
  - linux
categories: linux
---

# 1. shell 类型
系统启用什么样的shell取决于你个人的配置。在/etc/passwd文件中，第七个字段列出了默认的shell程序。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/shell-lijie.png)
查看一下/bin/bash 可以发现bash是可执行程序(有个星号，参见ls -F)。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/ls-bashshell.png)

再看看系统默认的shell，这是一个symbolic link 文件，指向dash。所以，默认和user交互的shell是bash shell；默认系统shell是dash shell。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/bin-sh-ls.png)

# 2. shell的父子关系


# 3. shell 内建命令
