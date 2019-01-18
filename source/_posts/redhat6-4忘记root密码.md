---
title: redhat6.4忘记root密码
date: 2019-01-14 10:47:13
tags: Linux
categories: Linux
---

# 1 进入 grub 引导程序  
重启进入 GNU GRUB 界面。  

# 2 用单用户模式登录 
输入 'e'字符，接着的界面会有3个选项出现：  
root (hd0,0)  
kernel /vmlinuz-2.6...  
initrd /initramfs-2.6...  

选中 "kernel" 这一行，然后按下"e"键，出现以下页面：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-14_105447.png)  

在这行的末尾输入 " single" (single前面有个空格)，回车。  

这时第二项依旧处于选中状态，然后按下 'b' 重启。  
操作系统启动完成，不需要输入密码，直接登录 root 账户。  
