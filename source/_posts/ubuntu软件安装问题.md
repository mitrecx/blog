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
最后还有一句：  
<font color=red>
E: Problem with MergeList /var/lib/apt/lists/ppa.launchpad.net_chris-lea_node.js_ubuntu_dists_xenial_main_binary-amd64_Packages  
</font>
这无疑是 __软件源__ (<font color=orange>/etc/apt/sources.list</font>)出问题了。  

解决办法：  
```sh
#删除apt-get install 的所有软件状态包
sudo rm /var/lib/apt/lists/* -vf
#更新并覆盖所有以apt-get 方式安装的软件源或包
sudo apt-get update
```
---------  

# /var/lib/apt/lists
/var 包括系统运行时要 **改变** 的数据。  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/java-%E8%BE%BE%E5%86%85%E8%A7%86%E9%A2%91%E5%AD%A6%E4%B9%A0/linux_var.png)  

如果/usr是 **安装时** 会占用较大硬盘容量的目录，那么/var就是在系统 **运作后** 才会渐渐占用硬盘容量的目录。   

因为/var目录主要针对常态性 **变动的文件** ，包括缓存(cache)、登录档(log file)以及某些软件运作所产生的文件。  

<font color=green>/var/lib/</font> 程序本身执行的过程中，需要使用到的数据文件放置的目录。  
**在此目录下各自的软件有各自的目录。**   
举例来说，MySQL的数据库放置到/var/lib/mysql/而rpm的数据库则放到/var/lib/rpm去。  

<font color=green>/var/lib/apt</font> 可以理解为apt的缓存。 安装软件Source list的问题都可以试试先删除缓存，再更新：  
```bash
sudo rm /var/lib/apt/lists/* -vf
sudo apt update
```

# <font color=orange>/etc/apt/sources.list</font> 的解读  
/etc/apt/sources.list 和/etc/apt/sources.list.d/\*.list的各文件。 是 **包管理工具 apt** 所用的记录软件包仓库位置的配置文件。  
<font color=orange>/etc/apt/sources.list</font>文件内容截取：  
```sh
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
# deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
#deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates universe

```
可以看到，每一行(条目) 由 4个字段组成：  
* 档案类型  
* 仓库地址
* 发行版
* 软件包分类  

## 档案类型 (Archive type)
条目的第一个词 deb 或是 deb-src 表明了所获取的软件包档案类型。  
档案类型有两种：   
1. __deb__ 档案类型为 __二进制预编译软件包__ ，一般我们所用的档案类型。  
2. __deb-src__ 档案类型为用于编译二进制软件包的源代码。  

## 仓库地址 (Repository URL)
条目的第二个词则是软件包所在仓库的地址。我们可以更换仓库地址为其他地理位置更靠近自己的镜像来提高下载速度。  
我用的镜像地址是 __阿里云__ 的。  

## 发行版 (Distribution)
跟在仓库地址后的是发行版。  
发行版有两种分类方法:  
一类是 **发行版的具体代号**，如 **xenial**, trusty,  precise 等；  
一类则是 **发行版的发行类型**，如oldstable, stable, testing 和 unstable。  

## 软件包分类 (Component)
跟在发行版之后的就是软件包的具体分类了，可以有一个或多个。  

Ubuntu 对软件包的分类可以用下表来表示(参考自 [Wikipedia](https://en.wikipedia.org/wiki/Ubuntu_%28operating_system%29#Package_classification_and_support)):  

||自由软件|非自由软件|
|---|----|--------|
|官方支持的|Main|Restricted|
|非官方支持的|Universe|Multiverse|

universe 社区维护的自由软件。  
multiverse 非自由软件。  

----
# References:  
1. [/etc/apt/sources.list 详解](https://blog.csdn.net/gong_xucheng/article/details/53886271)   
2. [深入理解linux系统的目录结构](https://www.jb51.net/LINUXjishu/151820.html)
