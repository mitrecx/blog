---
title: vim tutor
date: 2018-01-22 19:14:18
tags: 编程工具

---

# vim tutor

今天man vim 把vim使用手册祥读了一番，有一些新的发现。  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/man-vim.png)  
```shell
man vimtutor
```

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/man-vimtutor.png)  
发现一个tutor文件  

用vim打开这个文件  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/vim-vimtutor.png)  

这个tutor概括了一些vim使用方法，概括的简洁明了，很好。  

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/vim-tutor-lesson2.png)  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/vim-tutor-lesson4.png)  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/vim-tutor-lesson5.png)  
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/vim-tutor-lesson6.png)  

# tutor 之外
" . "是一个重复命令。  
" . "用于重复<font color=orange>输入模式</font>或<font color=orange>命令模式</font>中的命令。  
例如：  
4dd删除4行，然后按" . "会继续删除4行；  
/(查找) , n(查找下一个) , .(重复上一个编辑命令)这三个命令构成了奇妙的三重唱，如：/int(查找int)，用cw替换int输入double，然后按n(查找下一个),再按. ，下一个int直接变成double，不用再用键盘输入double了。

----
ps.  
vim里有3个操作符(operator)： c(change)，d(delete)，y(yank,copy)  
操作符常与其他命令(motion)组合使用，常用的motions：   
$(行末)  
G(文件末) -- d30G删除光标到第30行的内容    
w(操作一个词)  
e(end of word,和w差不多)  
/(操作到第一次匹配到的内容那一行（不包括）) -- ?反向  

一个操作符自身重复时（cc，dd，yy），仅对当前行起作用。可prepend数字，以操作多行。  

----  
:e FILENAME -- （需要保存当前文件，）自动退出文件，打开名为FILENAME的文件  

：r FILENAME -- retrieving and mering，可以配合!(ex模式下的exclamation point)，如： ":r !ls -al"  

**配置vimrc**：  
```sh
vim ~/.vimrc
```
添加以下内容：  
```sh
#显示行号
set nu
#自动语法高亮
syntax on
# 缩进
set autoindent
set smartindent

#匹配高亮
set hlsearch
# 默认缩进4个空格
set shiftwidth=4
#使用tab时 tab空格数
set softtabstop=4
#tab 代表4个空格
set tabstop=4
#使用空格替换tab
set expandtab
# 根据 /usr/share/vim/vim74/colors，更改配色方案，例如：
color murphy
```
