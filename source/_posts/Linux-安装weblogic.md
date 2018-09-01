---
title: Linux 安装weblogic
date: 2018-09-01 21:13:13
tags: weblogic
categories: weblogic
---
# Ubuntu16.04 上搭建 weblogic 应用服务器
weblogic简介：  
Oracle WebLogic Server is a scalable, enterprise-ready Java Platform, Enterprise Edition (Java EE) application server.  
 The WebLogic Server infrastructure(基础设施) supports the deployment(部署) of many types of distributed applications(分布式应用程序) and is an ideal foundation for building applications based on Service Oriented Architectures (SOA).  
Oracle WebLogic Server是一个可扩展的企业Java平台，企业版（Java EE）应用服务器。  
WLS设施支持的许多类型的分布式应用程序的部署，它是一个基于面向服务的架构（SOA）的理想的构建应用的基础。  
**简单来讲：WLS是一个企业级的应用服务器。**  

## 1. 准备工作 -- java开发环境
```sh
java -version
```
如果没有jdk环境的话，首先安装jdk。  
安装方法：
```sh
sudo add-apt-repository ppa:webupd8team/java

sudo apt update

sudo apt install oracle-java8-installer
```
如果不能使用 **add-apt-repository** ，那么需要安装：  
```sh
apt-get install python-software-properties
apt-get install software-properties-common
```
## 2. 安装weblogic
### 2.1 安装
[下载weblogic](http://www.oracle.com/technetwork/middleware/weblogic/downloads/index.html).  下载选择 generic installer(通用下载器) 即可。  

其实安装WebLogic Server 非常简单。由于你对 **Linux 图形桌面环境** 不了解，那么你可能会走一点弯路。    

工作需要，我买了阿里的云主机MITRE(Ubuntu16.04)。ssh 连接到MITRE上，然后在终端操作安装weblogic，可是，总是报 <font color=red>X11</font> 与图形桌面相关的错误。  
明明是按照Oracle官网的安装教程走的，不可能错的啊！！  
于是在自己的laptop(Ubuntu16.04)上也安装了一遍。第一步就出问题了！  
查看相关资料验证了我的猜想，**[Oracle](https://www.oracle.com/index.html) 的WebLogic Server安装必须要有桌面环境**。  

[远程桌面Ubuntu16.04 的方法，参考这里](https://mitrecx.github.io/2018/09/01/%E8%BF%9C%E7%A8%8B%E6%A1%8C%E9%9D%A2Ubuntu/)

执行 jar文件 开始安装，基本上全选默认:   
```sh
java -jar fmw_12.2.1.3.0_wls_generic.jar
```

[安装WebLogic Server细节问题，参考这里](https://docs.oracle.com/en/middleware/lifecycle/12.2.1.3/wlsig/installing-oracle-weblogic-server-and-coherence-software.html#GUID-E4241C14-42D3-4053-8F83-C748E059607A)

### 2.2 建域
创建一个域(domain)：  
```sh
# weblogic 12c 建域的脚本在这个目录下 /Oracle/Middleware/Oracle_Home/oracle_common/common/bin
# 其他版本的weblogic ，查找建域脚本所在目录
find . -name "config.sh" -print

# 打开 find 到的那个目录
cd config_dir

#执行建域脚本
./config.sh
```

**注1** : 需要输入密码，模式选product 而不选 development。    

**注2** : weblogic 12c 在 Middleware/wlserver 目录下也有一个 config.sh , 但这个目录下的脚本是不建议( **deprecated** )使用的。可以自行打开查看。    


**注3** :**<font color=pink>我这里（WebLogic 12c Release）创建域也要桌面环境，但 11g 是不用的。</font>**。  

### 2.3 配置密码文件 boot.properties
每次在终端启动WebLogic的时候都要输入用户名、密码，很麻烦。  
可以在AdminServer 目录下创建security/boot.properties  
boot.properties里只要写上用户名、密码就可以了。
```sh
# domain/base_domain/servers/AdminServer/security/boot.properties
username=weblogic
password=password
```
服务器启动一次，boot.properties会自动加密。  

## 3. weblogic 相关问题

./startWeblogic 启动webLogic， 启动到 **<Info\><Management\> <BEA-141107\>** 卡住不动，  
[据说](https://blog.csdn.net/weiruoao/article/details/46349359)，这是JDK一个bug。    
解决办法: 在weblogic启动脚本里 **setDomainEnv.sh**, 加入以下内容:   
```sh
JAVA_OPTIONS="${JAVA_OPTIONS} -Djava.security.egd=file:/dev/./urandom"
export JAVA_OPTIONS
```





## references
1. [WebLogic Reference Guide](https://docs.oracle.com/cd/E24902_01/doc.91/e23434/install_config_12_1_3.htm#EOHLU214)
