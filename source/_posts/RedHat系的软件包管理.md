---
title: RedHat系的软件包管理
date: 2019-01-15 15:43:59
tags: Linux
categories: Linux
---

耳濡目染，不学以能。  ---韩愈

----

# 1 Linux 包管理概述  
Linux 系统按照 包管理系统 可划分为两个派系： RedHat系 和 Debian系。   

RedHat 系(RedHat, CentOS, Fedora, Oracle Linux等)：  
1、 包格式： **.rpm**   
2、 包管理工具： **<font color=red>rpm</font>**，**<font color=red>yum</font>**，dnf(Fedora在yum之后开发的包管理工具)   
**rpm (RPM Package Manager)** 是一个包管理系统。  
**yum (Yellowdog Updater, Modified)** 是一个基于 rpm 的 包管理程序。  
**dnf (Dandified YUM)** , 最近的 Fedora 版本中，dnf 取代了 yum。dnf 是 yum 的一个现代化的分支，它保留了大部分 yum 的接口。  

Debian 系(Debian, Ubuntu等)：  
1、 包格式： **.deb**  
2、 包管理工具：  **<font color=orange>dpkg</font>**, apt-cache, apt-get, apt-config, **<font color=orange>apt</font>**  
**APT (Advanced Package Tool, 高级软件包工具)** 是一个 软件接口。  
**dpkg (Debian Package)** 是一个底层的包管理工具。  
**apt** 是一套处理包的工具集 (包含命令 apt)。  
**apt-cache, apt-get, apt-config 命令** 是 apt 命令 的超集 。  


# 2 RPM Package Manager

rpm 命令的用法有3大类：查询和校验包，安装更新和删除，杂项。  

## 2.1 QUERYING AND VERIFYING PACKAGES:
```
rpm {-q|--query} [select-options] [query-options]

rpm {-V|--verify} [select-options] [verify-options]
```
select-options, query-options, verify-options, install-options 详细参数参考 <code>man rpm</code>。   
常用参数：  
```sh
# 查询 所有已经安装的软件包
rpm -qa
rpm -q --all

# 查询 指定软件包信息
rpm -qi yum
rpm -q --info yum

# 查询软件包相关的文件
rpm -ql yum
rpm -q --list yum
```

## 2.2 INSTALLING, UPGRADING, AND REMOVING PACKAGES:
```
rpm {-i|--install} [install-options] PACKAGE_FILE ...

rpm {-U|--upgrade} [install-options] PACKAGE_FILE ...

rpm {-F|--freshen} [install-options] PACKAGE_FILE ...

rpm {-e|--erase} [--allmatches] [--justdb] [--nodeps] [--noscripts]
        [--notriggers] [--test] PACKAGE_NAME ...
```
常用参数：  
```sh
# 安装软件包. -v 表示 verbose 冗长输入，-h 表示 hash 井号进度条
rpm -ivh xxxx.rpm

# --nodeps  不验证软件包的依赖(no dependencies)
# --force 强制安装，即使覆盖其他包的文件也要安装
#强制安装
rpm -ivh --nodeps xxxx.rpm
#强制卸载
rpm -e --nodeps --force xxxx.rpm
```

## 2.3 MISCELLANEOUS:
```
rpm {--querytags|--showrc}

rpm {--setperms|--setugids} PACKAGE_NAME ...
```

## 2.4 软件包的命名
如下有 3 个 rpm 软件包：  
```
yum-  3.4.3-161.  el7.  centos.  noarch.rpm
yum-metadata-parser-  1.1.4-10.  el7.  x86_64.rpm
yum-plugin-fastestmirror-  1.1.31-50.  el7.  noarch.rpm
```
**rpm 软件包命名规则**：  
软件名-主版本号.次版本号-修订号. 适用的Linux平台. 适用的硬件平台.rpm  

Linux平台有： centos、el系列(RHEL/centos)。   
硬件平台有：i386、i686、x86_64、noarch(no architecture, 所有硬件平台通用)。  

# 3 yum
rpm 虽然可以安装软件包，但是需要自己安装 **依赖包**。安装依赖 是很一件繁琐的事(需要一个一个一层一层的下载，再安装)。  

通常情况下，安装软件都是用 **yum**，而不是 **rpm**。  

## 3.1 yum 简介
**yum 可以自动 管理 包间的依赖关系**。和 Debian系 的Advanced Package Tool (APT) 类似，yum与软件仓库(software repositories)一起工作，这些软件仓库可以在本地访问 也能通过网络连接访问。  

## 3.2 yum 工作原理
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/yum.png)  
这里只关注客户端，不考虑服务端。  

客户端的配置文件：<font color='#3366ff'>/etc/yum.conf</font> 和 <font color='#3366ff'>/etc/yum.repos.d/\*.repo</font>。  
<font color='#3366ff'>/etc/yum.conf</font> 包含一个 [main] section，可以通过它设置 全局作用的 yum 选项。  
<font color='#3366ff'>/etc/yum.conf</font> 文件还可以包含一个或多个 [repository] sections，可以通过他们设置 repository-specific options(特定的仓库选项)。但是，推荐的做法是 在<font color='#3366ff'>/etc/yum.repos.d</font> 目录下 新建 .repo 文件， 把 特定的仓库选项 放在 .repo 文件中。  

<font color='#3366ff'>/etc/yum.conf</font> 文件的 [main] Options 默认设置：  
```sh
[main]
# 存储 yum缓存和数据库文件 目录的绝对路径
cachedir=/var/cache/yum/$basearch/$releasever
# 安装成功后是否保存缓存：0-不保留，1-保留
keepcache=0
# 显示 调试输出： level 取值0到10，值越大debug输出越详细，0-不输出，2-缺省
debuglevel=2
# yum 日志文件 的绝对路径
logfile=/var/log/yum.log
# 确切的architecture ： 0-更新包时不考虑确切的 architecture，1-更新时考虑  
exactarch=1
# 废弃处理逻辑(obsoletes processing logic): 0-禁用，1-enable( 取决于包的spec file内容)
obsoletes=1
# GPG signature 检查(一种加密软件套件)： 0-disable，1-enable
gpgcheck=1
# yum plug-ins globally(yum全局插件): 0-disable, 1-enable
plugins=1
# installonlypkgs directive 用到的参数
installonly_limit=5
# 后两个作用不详
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```

仓库配置文件 <font color='#3366ff'>/etc/yum.repos.d/\*.repo</font> ：  
```sh
# 仓库的ID，可以取任意名字，只要不和其他的ID冲突即可  
[Repository_ID]

# a human-readable string describing the repository.  
name=repository_name
# 服务端 repodata文件夹 所在目录的URL，客户端安装包时访问的路径  
baseurl=repository_url
# 镜像服务器的地址列表. 注：baseurl 和 mirrorlist 只能选其一
mirrorlist=repository_url_list

# 1-Include this repository as a package source(将此仓库作为包源，支持HTTP、FTP、本地库).  
enabled={1|0}

# 是否进行 gpg签名(GNU Privacy Guard)检测，0-disable，1-enable(默认)。若启用gpg检查，需要告知其key是什么
gpgcheck={1|0}   
# 如果启用 gpg签名检测，则需要指定gpgkey的路径，次路径可以是远程服务器上的，也可以是本地的.
gpgkey=url
```

## 3.3 yum 命令 GENERAL OPTIONS

### 3.3.1 安装和删除
```sh
yum -y install package-name;

yum -y remove package-name;
```
-y 自动确认安装/删除    

例如安装 sl：  
```sh
yum -y install sl;
```

### 3.3.2 更新和升级
```sh
yum update package-name;

yum upgrade package-name;
```

### 3.3.3 缓存
清除缓存：
```sh
yum clean [headers|packages|metadata|dbcache|plugins|expire-cache|all]
```
packages 清除缓存目录下的软件包, 清空的是(/var/cache/yum)下的缓存。  
headers 清除缓存目录下的 headers。    
all 清除所有的 yum缓存。  

将服务器软件包信息缓存至本地：  
```sh
yum makecache;
```

### 3.3.4 查找和显示
```sh
# 显示 软件包信息
yum info sl

# 显示 软件包安装信息
yum list sl

# 查询 软件包依赖信息
yum deplist sl
```

# 4 RedHat6.4 重新安装 yum
RedHat6.4 原生 yum 需要注册付费才能下载软件，要想不注册，需要重新安装 yum ( CentOS6 版本 )。  

1、 卸载 RedHat6.4 原生 yum:  
```sh
rpm -qa | grep yum | xargs rpm -e --nodeps
```

2、 从 [网易镜像网站](https://mirrors.163.com/centos/6/os/x86_64/Packages/) 下载如下几个包(wget 下载过程略) 并安装：  
```sh
rpm -ivh python-iniparse-0.3.1-2.1.el6.noarch.rpm  
rpm -ivh python-urlgrabber-3.9.1-11.el6.noarch.rpm
rpm -ivh yum-metadata-parser-1.1.2-16.el6.x86_64.rpm

# 以下两个包要一起安装
rpm -ivh yum-3.2.29-40.el6.centos.noarch.rpm yum-plugin-fastestmirror-1.1.30-14.el6.noarch.rpm
```

**注意：**   
如果安装过程中有报 <font color=red>conflict 问题</font>，用 <code>rpm -Uvh xxx.rpm</code> 升级，或 用 <code>--force</code>强制安装。   
如果报 <font color=red>xxx needed by 问题</font>，那么需要下载 xxx.rpm 包并安装。  


3、 [下载 CentOS6-Base-163.repo](https://mirrors.163.com/.help/centos.html):  
```sh
cd /etc/yum.repos.d
wget https://mirrors.163.com/.help/CentOS6-Base-163.repo
```
把 CentOS6-Base-163.repo 文件中的 $releasever 替换 为 6。  
<font color=red>不要修改 epel.repo 文件</font>。  

4、 清理缓存：  
```sh
yum clean all
yum makecache
yum update
```

5、 完成，测试以下：  
```sh
yum install -y gcc
yum -y install sl
yum -y install tree
yum -y install htop
```

# 参考
1 [Linux 包管理基础：apt、yum、dnf 和 pkg](https://linux.cn/article-8782-1.html?pr)  
2 [CONFIGURING YUM AND YUM REPOSITORIES](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-configuring_yum_and_yum_repositories)  
3 [yum的工作原理以及如何建立yum仓库](https://blog.csdn.net/u012012939/article/details/48438103)  
4 [SETTING REPOSITORY OPTIONS](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-setting_repository_options)  
5 [yum 命令讲解](https://blog.csdn.net/shuaigexiaobo/article/details/79875730)  
6 [yum (software)](https://en.wikipedia.org/wiki/Yum_(software%29)   
