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

## git eclipse中的项目：  

1. 在GitHub上建一个repository  
2. 本地项目右键, Team , share Project , Git , 创建本地repository(勾选use or create)  
3. 新建的本地仓库有 **NO-HEAD** 标识，要先把远程仓库pull到本地仓库，使用 https协议  
4. commit 到本地仓库  
5. push 到 GitHub远程仓库  

这是我git的[结果](https://github.com/mitrecx/youranAccount)，  
注意一下.gitignore , 这个文件里是提交时指定忽略的文件。  
git 相关学校 [看这篇博客](https://mitrecx.github.io/2017/11/26/git%E7%AE%80%E5%8D%95%E6%95%99%E7%A8%8B/).  
