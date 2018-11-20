---
title: 理解shell
date: 2017-11-30 18:06:57
tags:
  - Linux
categories: Linux
---

# 1. shell 类型
系统启用什么样的shell取决于你个人的配置。在/etc/passwd文件中，第七个字段列出了默认的shell程序。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/shell-lijie.png)
查看一下/bin/bash 可以发现bash是可执行程序(有个星号，参见ls -F)。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/ls-bashshell.png)

再看看系统默认的shell，这是一个symbolic link 文件，指向dash。所以，默认和user交互的shell是bash shell；默认系统shell是dash shell。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/bin-sh-ls.png)

# 2. shell的父子关系
在bash shell里打开一个dash shell （子shell）
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/chile-shell.png)
可以看第二次执行ps -f 时，多了一个子shell。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/child-shell2.png)

进程列表：(pwd;ls;cd /ctc)，在括号里写命令，分号分隔，就是进程列表。进程列表会在子shell中执行。

# 3. shell 内建命令
內建命令和外部命令执行的时候有很大的不同。

外部命令又称文件系统命令。是存在与于bash shell 之外的程序。外部命令通常位于/bin, /usr/bin, /sbin, /usr/sbin中。可以用which和type找到它们的位置。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic2/bultin-cmd.png)
<font color='orange'> 当外部命令执行时，会创建出一个子进程。这种操作叫衍生（forKing）。</font>
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic2/shell-forking.png)
ps的父进程PID是9943。  

有的命令有多种实现，比如echo和pwd，可以用type -a 查看。
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic2/bultin-cmd-pwd.png)    

內建命令，不需要通过衍生出子进程来执行，也不用打开程序文件。所以，执行速度快，效率高。內建命令有很多: cd, fg, bg, history, test, alias...
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic2/bultin-cmd-alias.png)
