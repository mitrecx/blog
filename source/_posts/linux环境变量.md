---
title: Linux环境变量
date: 2017-12-01 16:07:55
tags:
  - Linux
categories: Linux
---

# 什么是环境变量
bash shell用一个叫做<font color=green>环境变量(environment variable)</font>的特性来存储有关shell会话和工作环境的信息。

在bash shell中，环境变量有两种：全局变量，局部变量。

## 全局环境变量
全局变量可以用于进程的所有子shell。
```bash
#查看全局变量
env
#显示变量的值
echo $HOME
#使用
ls $HOME
```
## 局部环境变量  
局部变量只能在定义它们的进程中可见。
```bash
#没有命令可以直接查看局部变量
#set 会显示所有环境变量(局部，全局)和用户定义变量
set
```

# 定义和删除变量
可以在bash shell中直接设置自己的变量。
```bash
#设置一个局部环境变量
my_variable="hello world"
#把局部变量变成全局变量
export my_variable
#也可以直接 export my_variable2="don't panic"

#删除my_variable
unset my_variable
```
注意：  
在子shell中修改全局变量的值，只在子shell中有效，在全局中，全局变量不会被改变。  
在子shell中删除全局变量，同样只对子shell有效。  

# 系统环境变量

## PATH
当在shell命令行输入一个外部命令时，shell必须搜索系统找到对应的程序。PATH环境变量**定义了用于进行命令和程序查找的目录**。  
可以修改PATH，加入新的程序的位置。  
** 对PATH的修改只能持续到退出或重启系统**

## 定位系统环境变量
当你登入linux系统启动一个bash shell时，默认情况下bash会在几个文件中查找命令。这些文件叫做<font color=green>启动文件</font>。  
bash 检查启动文件取决于你启动bash shell的方式。启动bash shell的3中方式：  
* 登录时作为默认登录shell
* 非登录的交互式shell
* 运行脚本的非交互shell   

### 登录shell
登录Linux系统时，bash shell作为登录shell启动。登录shell会在5个不同的启动文件中读取命令：
* __<font color=red>/etc/profile</font>__
* $HOME/.bash_profile
* <font color=red>$HOME/.bashrc</font>
* $HOME/.bash_login
* <font color=red>$HOME/.profile</font>  
(红色为我的系统Ubuntu的启动文件)  


  <font color=red>/etc/profile</font>文件是系统默认的bash shell 主启动文件。系统的每个用户登录都会执行这个启动文件。另外四个是针对用户的，可根据个人需求定制。  
打开<font color=red>/etc/profile</font>可以看到,它用到了__/etc/profile.d__目录下的所有文件。    


$HOME目录下的启动文件  
- $HOME/.bash_profile    
- <font color=red>$HOME/.bashrc</font>   
- $HOME/.bash_login   
- <font color=red>$HOME/.profile</font>  

用户可以编辑这些文件，添加自己的环境变量。这些环境变量会在启动bash shell会话时生效。  
### 交互式shell
检查HOME目录下 .bashrc 。
### 非交互式shell  
系统执行的shell脚本就是这种shell。就是脚本---没有CLI提示符。脚本用到的变量都是从父shell继承来的。  


## 环境变量持久化
把环境变量放在文件中，虽然可以放在/etc/profile中。但是这样不规范，我们应该在/etc/profile.d中建立.sh文件，把自己设置的全局环境变量放在里面。    

# 数组变量
```bash
#创建数组变量
mytest=(one two three four)

echo ${mytest[2]}
#输出three
```
