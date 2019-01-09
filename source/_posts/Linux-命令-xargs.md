---
title: Linux 命令 xargs
date: 2019-01-09 14:53:40
tags: Linux
categories: Linux
---
在 Unix 和 类Unix 系统中，**<font color=red>xargs</font>** 的作用是 **把 <font color=green>标准输入</font>(standard input) 转换成 其他命令的 <font color=green>命令行参数</font>(command-line arguments)**。  

**<font color=green>标准输入</font>**： 程序运行时从 **键盘输入**，或者 **管道** 传入的数据。   
例如，grep 从 **标准输入**(管道) 中 获取数据 并查找 root 所在行：  
```sh
cat /etc/passwd | grep "root"
```

**<font color=green>命令行参数</font>**： 此参数(数据) 在输入命令时就确定了。   
例如， grep 从 **命令行参数 /etc/passwd** 中获取数据 并查找 root 所在行:   
```sh
grep "root" /etc/passwd
```

# 1 为什么需要 xargs 做转换 ?  

上面的介绍中，grep 可以从 **标准输入** 获取数据，也可以从 **命令行参数** 获取数据。  

事实上，**所有 Linux 命令 都可以从 命令行参数 获取数据** (如果有命令行参数的话)。  
**只有一部分 命令 能够从 标准输入 中获取数据**，grep 和 cat 就在这一部分命令之中。  

举个例子， 文件 1.txt 内容如下：  
```
1.txt
hello
world
```
执行以下命令，观察结果 (结果是每条命令下的注释) ：   
```sh
echo "hello"
# hello

# cat 从标准输入中获取数据
echo "hello" | cat
# hello

# cat 同时标准输入和命令行参数获取数据，cat 会忽略 标准输入
echo "hello" | cat 1.txt
# 1.txt
# hello
# world

# cat 同时标准输入和命令行参数获取数据，- 使 cat 不忽略 标准输入
echo "hello" | cat 1.txt -
# 1.txt
# hello
# world
# hello
```
同样的，grep 的情况如下：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/xargs-hello.png)  

还有一些命令不能像 grep 和 cat 这样，支持 **标准输入**。   
比如 kill , rm , cp , echo 等 命令 就 **<font color=red>不支持 标准输入</font>**。  
而往往我们会有下面这种需求：  
```sh
# 查找进程号，通过管道传给 kill 杀死该进程
# 以下命令无法执行，因为 kill 不支持 标准输入
ps -ef | grep "weblogic.Server" | awk '{print $2}' | kill -9
```
这些 不支持标准输入的命令 想要 从标准输入中获取数据，就需要 **xargs** 命令做转换：  
```sh
# 以下命令可以正常执行
ps -ef | grep "weblogic.Server" | awk '{print $2}' | xargs -I {} kill -9 {}
```  

# 2 xargs 常用选项
##  -I 选项 (小写的 i)
用法：
```sh
xargs -I {} commandX {}
# 或者
xargs -I{} commandX {}
```
**{}** 表示从 标准输入 传入的数据，也可以换成别的字符， 如 %，a，b，c 等等。   


举例，获取 1.txt 文件的最后两行，并以 <code>行内容.txt</code> 为文件名 创建两个文件：  
```sh
tail  -2  1.txt | xargs -I{} touch {}.txt
```
## -p 选项
p 的含义 是 prompt 。  
执行命令前会输出 命令，并需要确认才能执行(y执行，其他不执行)。  
```sh
tail  -2  1.txt | xargs -p -I{} touch {}.txt
# touch hello.txt ?...n
# touch world.txt ?...n
```

## -d 选项
d 的含义 是 delimiter(分隔符) 。  
xargs 命令 默认的 分隔符是 空白字符(空格，tab，换行)， 可以用 -d 选项自定义分隔符。  

例如：  
```sh
echo "aa|bb|cc|dd" | xargs echo
# aa|bb|cc|dd

echo "aa|bb|cc|dd" | xargs -d"|" echo
# aa bb cc dd

# -n1 选项的含义是 每个字段作为 一行处理
echo "aa|bb|cc|dd" | xargs -d"|" -n1 echo
# aa
# bb
# cc
# dd

```

## -n 选项
用法：  
```sh
xargs -n数字 commandX
# 或者
xargs -n 数字 commandX
```
-n1 选项的含义是 每个字段作为 一行处理，    
-n2 选项的含义是 每2个字段作为 一行处理，  
-n3 选项的含义是 每3个字段作为 一行处理，  
...  

看一个例子：  
```sh
cat abc
# aa
# bb
# cc dd
# ee ff gg

cat abc | xargs echo
# aa bb cc dd ee ff gg

cat abc | xargs -I{} echo {}
# aa
# bb
# cc dd
# ee ff gg

cat abc | xargs -n1 echo
# aa
# bb
# cc
# dd
# ee
# ff
# gg

cat abc | xargs -n2 echo
# aa bb
# cc dd
# ee ff
# gg

cat abc | xargs -n3 echo
# aa bb cc
# dd ee ff
# gg

# -n 选项 和 -I 选项一起用，-n 选项会失效
cat abc | xargs -n1 -I{}  echo {}
# aa
# bb
# cc dd
# ee ff gg
```
注意： **<code>-n 选项</code> 和 <code>-I 选项</code> 一起用，-n 选项会失效**。  





# 参考
1、 [xargs-wiki](https://en.wikipedia.org/wiki/Xargs)  

2、 [xargs命令详解，xargs与管道的区别](https://www.cnblogs.com/wangqiguo/p/6464234.html)  

3、[Linux and Unix xargs command tutorial](https://shapeshed.com/unix-xargs/)  
