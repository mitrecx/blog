---
title: eclipse中使用git
date: 2018-10-09 11:12:53
tags: git
categories: git
---
# eclipse 使用 git 进行版本控制
如果没有git插件，需要先[安装git插件](https://www.eclipse.org/egit/download/)。    

git配置如下图：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/eclipse-git1.png)  

**这是我在 push 时遇到的三个错误**：  
1、 <font color=red>
Can't connect to any repository: https://github.com/mitrecx/youranAccount.git
(https://github.com/mitrecx/youranAccount.git: Premature EOF)  
</font>  

2、 <font color=red>
Can't connect to any repository: https://github.com/mitrecx/youranAccount.git
(https://github.com/mitrecx/youranAccount.git: Received fatal alert: bad_record_mac)  
</font>  

3、 <font color=red>Time out</font>  

前面两个问题应该是我在push之前，没有pull导致的。  
超时应该是没有配置.ignore, 提交了所有的文件(包括jar包)导致30s内没有push完。

**解决方法**：  
删除本地 **.git** 文件，然后按照以下步骤即可。  

## git eclipse中的项目：  

1. 在GitHub上建一个repository  
2. 本地项目右键, Team , share Project , Git , 创建本地repository(勾选use or create)  
3. 新建的本地仓库有 **NO-HEAD** 标识，要先把远程仓库pull到本地仓库，使用https协议  
4. commit 到本地仓库  
5. push 到GitHub远程仓库  

这是我git的[结果](https://github.com/mitrecx/youranAccount)，  
注意一下.ignore , 这个文件里是提交时指定忽略的文件。  
如果文件(夹)在.ignore 中，commit到本地仓库时，不会显示.ignore 中的文件。    
