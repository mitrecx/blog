---
title: Linux系统程序设计2-进程控制
date: 2018-02-14 21:37:12
tags: Linux
---

# 进程概述
内核为进程映像分配内存，但还会在大量寄存器中维护控制信息。内核利用控制信息来切换进程。  


## 虚拟地址空间
在执行C程序时，“程序加载器”把磁盘上的二进制代码转移到内存中。  
如果 **进程** 需要，内核还会在内存中生成额外的空间。**进程可以访问这一部分内存位置** 称为进程的<font color=orange>虚拟地址空间</font>。  
虚拟地址空间划分为许多段：  
- 文本段：包含要执行的指令(instruction)，是从磁盘文件读入的。一个程序的多个实例可以共享这个段（如：三个运行vi的用户将使用同一个文本段）。  
- 数据段：存储着程序使用的全局、静态变量。  
- 栈：存储着局部变量和函数的参数及返回地址。  
- 其他段：命令行参数和环境变量通常在栈的底部。  


## 进程表
每个活动的进程的属性都存储在一个结构中，这个结构代表 **进程表** 中的一项。  
现代Linux/Unix在内存中维护进程表。进程表的重要属性如下：  
- 进程的PID和PPID
- 进程的状态——运行、睡眠、僵尸等
- 进程的真实UID和GID
- 进程的事实UID和GID（effective 事实的，有效的）
- 文件说明符表
- 文件创建掩码
- CPU使用信息
- 挂起的信号掩码
- 信号处置表

一个fork子进程的 **进程表项** 有许多字段都是从其父进程复制而来的。  
当子进程执行一个新程序是，进程表项保留，但一些字段会发生变化。  
**<font color=orange> 进程表中包含了与信号相关的重要数据</font>**

# 进程环境
一个进程的环境包含存储在栈底部的shell **环境变量**。这些变量可以在全局 environ 变量中获得，在C程序中声明如下：  
```c
extern char **environ;
```
environ 是一个数组，其中的元素是指向char的指针（就是C风格字符串）。  
这个数组存储的指针指向 name=value 形式的环境变量字符串。这些值可以用两个函数getenv和setenv来设置和提取：  
```c
//由环境变量名提取其值value，返回指向char类型(value)的指针
char *getenv(const char *name);

//envname是变量名，envval是变量值，overwrite决定是否覆盖已有值(0不覆盖，1覆盖)
int setenv(const char *envname, const char *envval, int overwrite);
```
举例：  
```c
//获取PATH的值
char *path = getenv("PATH");

//修改PATH为.
setenv("PATH", ".", 1);
```
process.c查看一些进程属性：  
```c
#include <stdio.h>
#include <stdlib.h>

int main(void){
        printf("PID: %4d, PPID: %4d\n", getpid(), getppid());
        printf("UID: %4d, GID: %4d\n", getuid(), getgid());
        printf("EUID: %4d, EGID: %4d\n", geteuid(), getegid());
        printf("PATH = %s\n", getenv("PATH"));
        setenv("PATH",".",1);
        printf("New PATH=%s\n",getenv("PATH"));
        exit(0);
}
```
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/linux_process.png)  

# fork: 复制当前进程
通过fork()系统调用我们可以创建一个和当前进程印象一样的新进程.我们通常将新进程称为子进程,而当前进程称为父进程.  
而子进程继承了父进程的整个地址空间,其中包括了进程上下文,堆栈地址,内存信息进程控制块(PCB)等.  

fork的语法很简单，但其返回方式 **不同寻常** :  
```c
pid_t fork(void);
```
**为使内核能够区分原进程及复制进程**， fork返回两次，分别给出不同的返回值：  
* 子进程返回0
* 父进程返回子进程的PID  

![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/linux_fork.png)  
一个简单的fork程序：  
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
int main(){
        pid_t pid;

        printf("before fork \n");
        pid=fork();

        if(pid>0) //返回子进程pid，在父进程中
        {
                sleep(1);
                printf("parent-----pid:%d\t ppid:%d\t child pid: %d\n", getpid(), getppid(), pid);
        }
        else if(pid ==0) //返回0，在子进程中
                printf("child -----pid:%d\t ppid:%d\n",getpid(),getppid());
        else  //这是返回-1的情况
        {
                printf("fork error\n");
                exit(1);
        }

        //以下语句会输出两遍，子父进程各一遍
        printf("Both processes continue from here\n");
        exit(0);
}
```
![](http://mitre.oss-cn-hangzhou.aliyuncs.com/blog_pic5/linux_fork_2.png)  

以上程序中，sleep(1)是为了让父进程在子进程后执行。**因为子进程和父进程的执行顺序是随机的** 。  

fork()系统调用总结：  
1）fork()函数的一次调用返回2次返回。  
2）fork系统调用之后，父进程和子进程交替执行，并且它们处于不同空间中。  
3) 将fork()返回值大于零设置为父进程,这是因为子进程获得父进程的pid相对容易,而父进程获子进程pid难,所以在fork()系统调用中将子进程的pid作为父进程的返回值，解决了获得子进程pid的问题。  

# exec: 进程创建的最终步骤
我们执行 fork 操作以创建进程，但多半会在后面跟有 exec 操作，在子进程的地址空间中运行另外一个程序。  
exec叙述：  
-------
在 fork 期间继承的许多属性都不会因为 exec 而改变。例如：上一个程序的文件说明符、root目录、umask设置、全局变量都保持不变。  
不过coder可以通过两个方式来改变exec后的环境：  
- 关闭一个或多个文件说明符，这样，在 fork 前后打开的文件就不能直接在exec的进程中读取。  
- 向exec的进程中传送一个单独的环境，而不是在environ全局变量中保存默认环境。  

我们经常用 exec 这个名称来指代这种关系，并没有一个名为 exec 的系统调用。  
事实上，有一个execve，有五个库函数都是以她为基础的。我们直接将她们称为exec或exec系列。  
整个系列可以分为两个部分，分别为 execl 集和 execv 集。  
注：  
execl中的 ”l” 表示一个参数列表（list），而execv中的 “v” 表示参数数目可变（variable）。  

## 两个库函数：execl和execv
execl不使用PATH；第一个参数（path）表示一个绝对或相对路径名；其他参数表示命令行中的每个单词。  
execl用法：  
```c
int execl(const char *path, const char *arg0, ..., (char *) );
```
例子：  
```c
execl("/bin/wc", "wc", "-l", "foo", (char *) 0);
```
在例子中我们看到，execl的参数以 **(char \*) 0** 结尾，这就是一个0，被强制转换为指向char的指针，相当于NULL。  
例子表达的意思就是：  
```sh
wc -l foo
```

因为execl要求单独指定命令行的每个单词，所以如果 **在运行是才能知道参数** ，execl就无法使用了。解决办法是用execv。  
execv也不使用PATH，第一个参数（path）表示一个绝对或相对路径名；第二个参数是一个存放字符型指针的数组，数组里存放的是要执行的命令。  
execv用法：  
```c
int execv(const char *path, char *const argv[]);
```
例子：  
```c
char *cmdargs[]={"wc", "-l", "/etc/passwd", NULL};
execv("/bin/wc", cmdargs);
```
相当于shell中的：  
```sh
wc -l /etc/passwd
```

~~其实我并没有感觉execv有优越性。~~  
```c
#include <stdio.h>
int main(){
  execl("/bin/wc", "wc", "-l", "/etc/passwd", (char *)0 );
  printf("execl error");
}
```
为什么最后写一句”输出错误”？  
因为除非导致错误，否则对exec的调用不会返回。  

## 其他exec成员
execlp和execvp：  
这些函数使用PATH查找命令的位置，所以第一参数只要给出命令名即可。  
这里略过。  

# 收集退出状态：wait 和 waitpid
一般情况下，一个进程的终止要么是通过main，要么是调用exit或return。它向调用者返回退出状态，这一状态可以在shell中的参数 “$?” 中获得。  
当一个进程终止而又没有显式调用exit时，退出状态为 0 。在任何一种情况下，内核都会关闭所有已打开文件，刷新所有输出流，释放进程地址空间。**但进程没有被完全删除**。  
接下来，内核将退出状态放在进程表中，并将子进程的状态改为僵尸（zombie）。这一机制是“寄希望于” **父进程最终可能调用wait或waitpid**，以拾取退出状态。  
如果父进程最终这样做了，内核就释放该进程表项，并从系统中删除该进程。  
注：  
我们无法杀死一个僵尸，因为它不是进程，但太多的僵尸会消耗进程表的可用位置。ps输出内容在最后一列将僵尸显示为<defunct>。为清除僵尸，可能需要重启系统。  

## wait：父进程等待子进程死亡

## waitpid：更强大的等待机制
我不清楚为什么需要 wait和waitpid，如果是为了完全删除进程，显示调用exit不就可以了吗？  
以后理解再说明。  

----

[1]Sumitabha Das, Your UNIX/Linux: the Ultimate Guide, Third Edition
