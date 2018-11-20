---
title: No such file or directory
date: 2017-12-17 12:19:28
tags: Linux
categories: Linux
---
我已经下载了jdk8(Java SE Development Kit 8u151)，并配置好了环境变量，可是运行java时却提示，找不到文件。**文件确实是存在的，而且不是无效的符号链接文件。用file命令查看文件是可执行的。**<font color=orange>这报了一个不该报的错误啊？</font>    
[网上解决方法： ](https://stackoverflow.com/questions/9081962/java-is-installed-in-listing-but-execution-produces-java-no-such-file-or-d/9082947#9082947)   
说我的Ubuntu运行环境是64位，没有所需要的32位运行环境。没错，是我下载错了，我下载的是x86（[32位的jdk](http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-i586.tar.gz)），而不是[x64](http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz)。**如果我想在64位Ubuntu上安装32位jdk必须要有32位的环境。**  

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic4/%E6%89%BE%E4%B8%8D%E5%88%B0%E6%96%87%E4%BB%B6%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95.png)   
但是这样并不能很好的解决问题，在64位Linux系统上安装32位的软件确实是不妥当的。  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic4/%E6%B2%BB%E6%A0%87%E4%B8%8D%E6%B2%BB%E6%9C%AC%E5%95%8A.png)  
可以看出问题换了一种方式存在。  

还是下载x64解压，配置环境变量靠谱。  

还有一种更简单的 源[安装](https://www.cnblogs.com/a2211009/p/4265225.html) 的方式：
```shell
sudo add-apt-repository ppa:webupd8team/java

sudo apt-get update

sudo apt-get install oracle-java8-installer
```
