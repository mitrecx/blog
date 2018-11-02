---
title: 简单的bash shell
date: 2017-11-30 13:47:36
tags:
  - Linux
  - tech
categories: Linux
---

# 关于文件
<font color=green>pwd </font>: <font color=green>P</font>rint name of <font color=green>W</font>orking <font color=green>D</font>irectory

<font color=green>ls </font>:   
过滤器（通配符）：
- 问号（？）代表1个字符
- 星号（\*）代表0个或多个字符
- 元字符通配符[a-z]表示a-z中任一字符
- 元字符通配符[!a]表示匹配除a以外的字符

和SQL中的通配符类似，通配符和正则匹配的区别在与：通配符匹配所有内容。  
例如：
- 通配符匹配：‘my’表示匹配的内容全名就是‘my’
- 正则匹配：‘my’表示匹配的内容里有‘my’

```shell
ls -l my*  #ls 列出以my开头的文件信息
```

## 文件操作基本命令：
1. 操作
- touch
- cp
- mv 改名或移动
- rm

2.查看
- tree
- file 查看文件类型
- cat , head , tail
- more 显示文本文件
- less 是more的升级版，可以查找


## 链接文件：  
1. <font color=orange>硬链接</font>:  
hard link，硬链接和原始文件是同一个文件--唯一标识文件的inode号是一样的。

2. <font color=orange>符号链接</font>（软链接）:  
symbolic link，符号链接是一个单独的文件。这个文件指向原始文件（原始文件一定要存在）。ln -s 可以创建符号链接。

用 __ls -li__ 查看文件时，可以看到，软链接文件的大小比原始文件小，此外还有一个指针指向原始文件，inode号不同。硬链接和原始文件一模一样。  

**硬链接** 存在以下几点特性：  
* 文件有相同的 inode 及 data block  
* 只能对已存在的文件进行创建
* 不能交叉 **文件系统** 进行硬链接的创建
* 不能对 **目录** 进行创建，只可对文件创建
* 删除一个硬链接文件并不影响其他有相同 inode 号的文件

**符号链接** 特点：   
* 软链接有自己的文件属性及权限等
* 可对不存在的文件或目录创建软链接
* 软链接可交叉 **文件系统**
* 软链接可对文件或 **目录** 创建
* 创建软链接时，链接计数 i_nlink 不会增加
* 删除软链接并不影响被指向的文件，但 **若被指向的原文件被删除，则相关软连接被称为死链接** (即 dangling link，若被指向路径文件被重新创建，死链接可恢复为正常的软链接 )

# 关于进程
<font color=green>ps  </font>:  
process, 查看当前时间点的进程信息。  
ps有两个版本，但相差不大。给出两个版本最常用的参数。  
BSD版本：ps -aux
UNIX版本：ps -ef

<font color=green>top  </font>:  
task of process，top将输出进程叫做任务（task）。top可以<font color=red>实时监测</font>进程的状态。  
<font color=orange>建议使用htop</font>，top的升级版。

<font color=green>kill  </font>:  
kill杀死指定进程号进程。可以和对应的linux进程信号联用。比如：9无条件终止；1挂起；2中断等等。  
<font color=green>killall  </font>:  
killall杀死指定进程名进程。例如：  
```sh
killall http*  #杀死所有以http开头的进程
```

# 磁盘存储

<font color=green>df  </font>:  
<font color=red>D</font>isk space available on <font color=red>F</font>ile system，可以查看所有挂载的文件系统，以及磁盘空间的使用情况。  
一般用法：
```sh
df -h  
```

<font color=green>du  </font>:  
<font color=red>D</font>isk <font color=red>U</font>sage，磁盘使用情况。一般用于查看文件/目录大小。  
一般用法：
```sh
du -h  #当前目录下各个文件所占磁盘空间
```

<font color=green>mount  </font>:  
mount原意为骑上，增加。这里表示挂载。
用法：
```shell
#查看挂载情况，我觉得用df更好
mount  

#把U盘挂载/dev/sdb1挂载到/media/disk
mount -t vfat /dev/sdb1 /media/disk

#卸载U盘
umount /dev/sdb1

#下面命令可以修复Linux下的win盘挂载错误
ntfsfix /filesystem
#如果报错，关闭win快速启动
```

# 处理数据文件

<font color=green>sort  </font>:  
排序数据。  
options:
- -b  --ignore-leading-blank忽略起始空白
- -n  --numeric-sort
- -g  --general-number-sort把值当成浮点数
- -r  --revers

一个例子：
```shell
#当前目录下文件占用空间情况，逆序输出
du -sh * | sort -nr
```

<font color=green>grep  </font>:  
搜索数据--非常流行的命令。  
synopsis：  
  grep [options] pattern [file]  
  在名为 file 的文件中查找 含有pattern关键字的行。

options:

 - -v 相反匹配
 - -c 匹配到的总行数
 - [] 正则匹配指定字符，如[tf]匹配t或f
 - .
 - \*  
  等等

<font color=green>tar  </font>:  

gzip->.gz  
GNU压缩工具
```shell
#压缩：
tar –zcvf filename.tar.gz filename(dirname)
#解压：
tar –zxvf filename.tar.gz filename(dirname)
```
bzip2->.bz2
采用
```shell
#压缩：
tar –jcvf filename.tar.bz2 filename(dirname)
#解压：
tar –jxvf filename.tar.bz2 filename(dirname)
```
