---
title: Linux 处理中文乱码文件
date: 2019-01-08 15:14:39
tags: Linux
categories: Linux
---

# 1 问题描述
在 Windows10 系统中创建一个 包含中文的 文本文件 test.txt，  
把 test.txt 上传(通过 FlashFXP 5 或者 Xftp6 上传)到 Linux (Ubuntu16.04)，  
vim/cat/nl 打开 test.txt 文件乱码。  

# 2 问题排查
Win10 系统编码方式：  
```sh
chcp
# 936
```
代码页936 表示 gbk 编码。  
代码页65001 表示UTF-8 编码。  

Ubuntu16.04 系统编码方式：  
```sh
locale
# LANG=zh_CN.UTF-8
```

## 2.1 解决方式一
在 linux 系统中，对文件进行编码转换：  
```sh
# convert text from one character encoding to another 
iconv -f gbk -t utf-8 test.txt > target.txt
```
-f from-encoding 表示 原编码格式；  
-t to-encoding 表示 目的编码格式

**<font color=green>target.txt 文件内容 显示中文正常</font>**。  

## 2.2 解决方式二(无效)
修改 Win10 系统编码方式为 UTF-8：  
通过注册表编辑器 ( cmd -> regedit ) 修改
```
HKEY_CURRENT_USER\Console\%SystemRoot%_system32_cmd.exe
```
的 codepage 数值 为 65001 (十进制) 。重启计算机。  
然后重新上传文件到 Linux。  
亲测这个方式 **无法解决中文乱码问题**。  
