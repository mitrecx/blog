---
title: 再装Ubuntu
date: 2017-11-30 14:33:37
tags:
  - Linux
categories: Linux
---

由于之前遇到的Ubuntu16.04安装fcitx的问题没有解决，而后又遇到scim崩溃--把系统卡死（htop看到4核cpu和12G内存全部变红）。

我重新安装了Ubuntu，现在就做个简单的总结。

# 修复grub & 再添加引导win10
安装了Ubuntu16.04在win10上，没有grup的引导，开机直接进04在win10。  
解决方法：
利用Ubuntu安装U盘try Ubuntu来修复。连上网，打开终端：  
```shell
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update

#安装boot-repair
#这个工具可以修好大多数引导问题
sudo apt-get install -y boot-repair && boot-repair
```
然后出现boot-repair的界面，选择Recommend repair过几分钟就OK了。

重启以后，grub引导选项里发现没有win10 。
```shell
#更新一下grub就有win10 了
sudo update-grub  
#sudo update-grub2 #也行
```
问题已经没有了。
如果想排序grub引导os的顺序参考下图：  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic/grub-sort.png)
30_os-prober 那个就是win10 。把30改为08或09就能放在前面了。

注：  
1. 个人的一些个性化设置，如命令别名、路径等，可以在~/.bashrc中设置。
2.  

```shell
 locate grub | grep grub
 ```

可以看到关于grub的设置在/ect/default和/boot/grub中也有。


----
# grub BASH-like
在修复之前我还遇到一个问题：开机显示  
<font color=orange>Minimal BASH-like line editing is supported.</font>  
这是修复grub出错的一个正常提示。修复方法在上面已经给出了。  
下面给一个进入win10 的方法：
```shell
#从硬盘的第一个分区C盘启动
grub>root (hd0,0)
#指示grub读取分区第一个扇区的引导记录
grub>chainloader (hd0,0)+1
#启动grub
grub>安装boot
```
看似这样可以进入grub引导。实际上，会进入win的mbr引导。

----
# 搜狗

这次安装搜狗输入法就简单的多了。
1. 下载Linux版搜狗输入法
2. dpkg 命令安装deb软件包
3. 缺少依赖，sudo apt-get install -f 修复依赖问题
4. 再安装，完成。  

可以发现这里没有安装fcitx。但是还是装好了搜狗，重点在于install后面的 __<font color=red> -f </font>__  。

----
# 别的安装：

1 . Google浏览器翻墙在官网下载Linux安装包。  
2 . atom 在官网下载安装包。  
3 . shadowsocks-QT5安装方法：
网上的添加源安装行。也可[参考这里](https://github.com/shadowsocks/shadowsocks/tree/master)。安装完成，再在chrome上安装插件SwitchySharp，配置一下即可翻墙。
