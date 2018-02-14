---
title: Linux系统程序设计1--文件
date: 2018-02-14 19:48:53
tags:
---

# 1.  linux系统程序设计---文件
系统调用是一种例程。  
系统调用用于执行文件I/O、分配内存、创建进程等。  
标准ANSI C库函数，如fopen和printf可以作为对象模块捆绑到一个档案中。系统调用与之不同，它们内置在内核中，一般由汇编语言编写。但是，提供了接口，可以像调用函数一样调用接口。  
<font color=orange>库函数通常不包含系统调用提供的精细控制功能</font> 。例如：  
fopen不能设置文件权限，而open可以设置。  
只有利用stat系统调用才能知道文件的大小。  
只有fork系统调用才能创建进程。  

**系统调用**要比调用**库函数** 的的开销大.  

# 2.  errno 和 perror：处理错误
**系统调用在错误时返回-1**  
错误的发生可能会有多种原因——资源不可用、接收到一个信号、I/O操作失败、调用参数无效。  

当系统调用返回-1时，内核将静态（全局）变量errno设定为一个正整数。这个**用符号常量表示的整数**与**错误消息**相关联。  

仅当系统调用返回错误时才使用perror。  
perror函数语法：  
void perror(const char \*s)  ;
函数的以一个字符串s为参数。函数输出：打印该字符串s、一个冒号、一个空格，然后是与errno相关联的消息。  


|符号常量  | errno  | 消息      |
|---------|--------|----------|
|E PERM   | 1     |Operation not permitted|
|E NOENT  | 2     |No such file or directory|
|E SRCH   | 3     |No such process|
|E INTR   | 4     |Interrupted system call|  
|E IO     | 5     |I/O error|
|E ACCES  | 13    |Permission denied|
|E EXIT   | 17    |File exits|
|E NOTDIR | 20    |Not a directory|
|EISDIR   | 21    |Is a directory|


以下程序show_errors.c

```c
#include <fcntl.h>  //获取open中的O_RDONLY
#include <stdio.h>  //获取fprintf中的stderr
#include <stdlib.h> //获取exit
#include <sys/error.h>  //获取errno

int main(int argc, char **argv){
  //以只读模式打开
  if(open(argv[1],O_RDONLY)==-1) //O表示open，RD表示read。
  {
    fprintf(stderr, "errno=%d\n", errno);
    perror("open");
  }
  exit(0);
}
```
执行时，如：
$ ./a.out foofoo  
errno = 2  
open: No such file or directory

# 3. open: 打开和创建文件
一些与文件Ｉ/O相关的系统调用：　　
- open , close
- read , write
- lseek
- truncate , ftruncate

**在读写一个文件之前，必须要用open打开它。**  
open语法：   
int open（const char \*path, int oflag, ...)  ;  
**open以int形式返回一个文件说明符 (<font color=orange> file description</font> ) 。当发生错误时，open返回 -1 。**  
第一个参数(path)是一个指针，指向一个表示文件路径名的字符串(绝对路径或相对路径)，  
第二个参数(oflag)用于**设置打开模式**(读、写或读-写)。这一模式用三个符号常量表示：  
**O_RDONLY** 打开文件供读取  
**O_WRONLY**  打开文件供写入   
**O_RDWR**    打开文件供读和写  
这些常量在/usr/include下的文件 fcnlt.h 中定义。  
用法:  
```c
int fd;
//以只读方式打开passwd文件，如果打开出错，就打印错误
if( (fd=open("/etc/passwd",O_RDONLY)) == -1){
  perror("open");
  exit(1);
}
```

此外，open还有更多打开模式 ( 也在fcntl.h中定义 ):  
**O_APPEND**  追加模式打开文件(仅当打开模式为O_WRONLY或O_RDWR)   
**O_TRUNC**  将文件截短为0(条件同上)，其实就是覆盖文件(注:truncate  截短、缩短)  
**O_CREAT**  若文件不存在，则创建文件  
**O_EXCL**   若指定O_CREAT，且该文件存在，则生成一条错误  
**O_SYNC**  同步读写操作。确保在将数据写到磁盘之前，write不会返回  
```c
//类似shell中的>>
fd=open("foo.txt", O_WRONLY | O_APPEND )
//类似shell中的>
fd=open("../foo.txt", O_WRONLY | O_TRUNC )
```

如果文件不存在，就要用O_CREAT（有时会用到O_EXCL）创建它，并在第三参数指定其权限。  
**Linux/Unix在sys/stat.h中提供了符号常量以设置权限**  
但是也可以，用八进制(octal)数设置权限，必须添加前缀0。（例如0644而不是644）  
0644等同于：  
S_IRUSR|S_IWUSR | S_IRGRP | S_IROTH  
表示用户可读写，组可读，其他可写  
```c
//打开foo.txt，以截断方式写入，如果foo.txt不存在，就创建该文件
fd = open("foo.txt", O_WRONLY|O_CREAT|OTRUNC, 0644);
```

# 4. close: 关闭文件
**程序在终止之前会关闭所有已打开文件，但不在需要文件时主动关闭是一种好习惯。**  
close系统调用可以关闭一个文件:  
```c
int close(int fd);
```
close用文件描述符( file description )作为参数，实际上是释放**对该进程的文件说明符**。  
close成功返回0，失败返回-1.  

# 5. read:  读取文件
两个系统调用——read和write——负责对常规文件(REG,regular)以及管道和套接字进行读写操作。  
**这两个调用的语法类似，都利用了用户定义的缓冲区。**  
read利用open返回的文件说明符来读取文件：  
```c
ssize_t read(int fildes, void *buf, size_t nbyte);
```
read尝试从文件说明符fildes向缓冲区buf中读取nbyte个字符。  
**文件说明符**就是要**读取的对象**，**缓冲区**是读取后**内容存放的地方**，**nbyte**是一次**读取n个字节**  
注：nbyte通常是缓冲区本身的大小  
```c
#define BUFSIZE 4096
int n;
char buf[BUFSIZE];
while( (n=read(fd, buf, BUFSIZE))>0 ) //由open获得的fd
```
每次read调用，读取4096个字节。**在遇到EOF之前，read返回所读字符数，~~而遇到EOF时，返回0~~( 准确地说，是文件的大小是缓冲区的整数倍，最后一次遇到EOF返回0 )**。但最后一次迭代可能例外，最后一次迭代中，返回的是尚未读取的字符数（因为文件的大小可能不是缓冲区大小的整数倍）。read错误返回-1。  
例如：如上程序，fd描述的文件有4100个字节，第一次迭代read返回4096，第二次迭代read返回4个字节，第三次返回0。  

如果要处理所读取的每个字符，可将buf声明为一个char变量，然后将其地址传给read：  
```c
int n;
char buf;
while( (n=read(fd, &buf, 1))>0 )
```
用单字符缓冲区读取100个字符需要100次系统调用，这个代价高昂。在这种情况下，诸如fgetc之类的**库函数**是比较好的选择。  

# 6. write： 写文件
```c
ssize_t write(int fildes, const void *buf, size_t nbyte) ;
```
write把**缓冲区**的内容写入到fildes说明的**文件**中。  
write返回所写字符个数。如果write运行期间磁盘已满或文件大小超出系统限制，则write返回-1。  
```c
#define BUFSIZE 8192
int n;
char buf[BUFSIZE];
n = write(fd, buf, BUFSIZE);
```

和read一样，write可以一次写一个字符：  
```c
char buf;
write(fd, &buf, 1);
```

------------------
一个利用系统调用复制文件的程序：  

```c
//quit.h
#include<stdio.h>
#include<stdlib.h>

void quit(char *, int);
```

```c
//quit.c
#include "quit.h"

void quit(char *message, int exit_status){
        perror(message);
        exit(exit_status);
}
```


```c
//ccp.c
#include<fcntl.h>
#include<sys/stat.h>
#include "quit.h"
#define BUFSIZE 1024

int main(int argc,char **argv){
        int fd1,fd2;
        int n;
        char buf[BUFSIZE];

        if((fd1=open(argv[1],O_RDONLY))==-1)
                quit("open",1);
        if((fd2=open("passwd.bak",O_WRONLY|O_CREAT|O_TRUNC, 0664))==-1)
                quit("open2",2);

        while( (n=read(fd1,buf,BUFSIZE))>0){
                if(n!=write(fd2,buf,n))
                        quit("write",3);
        }
        close(fd1);
        close(fd2);
        exit(0);
}
```
编译、运行：  
```bash
#编译quit.c 生成quit.o目标文件
$ gcc -c quit.c

#用ccp.c和quit.o编译出a.out(默认的可执行文件名，可用-o指定别的名)
$ gcc ccp.c quit.o

#执行程序，复制passwd的内容到当前目录下的passwd.bak文件中
$ ./a.out /etc/passwd
```
write不一定写入磁盘，如果把ccp.c中的第二个open和close删除，把fd2改为1 ( 或改为STDOUT_FILENO，需要包含头文件unistd.h )，就可以写到标准输出了。
```c
#include<fcntl.h>
#include<sys/stat.h>
#include <unistd.h>
#include "quit.h"
#define BUFSIZE 1024

int main(int argc,char **argv){
        int fd1,fd2;
        int n;
        char buf[BUFSIZE];

        if((fd1=open(argv[1],O_RDONLY))==-1)
                quit("open",1);
//      if((fd2=open("passwd.bak",O_WRONLY|O_CREAT|O_TRUNC, 0664))==-1)
//              quit("open2",2);

        while( (n=read(fd1,buf,BUFSIZE))>0){
                if(n!=write(STDOUT_FILENO,buf,n))
                        quit("write",3);
        }
        close(fd1);
//      close(fd2);
        exit(0);
}
```
注：
读写标准流时，应当使用符号常量STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO作为文件说明符，而不是他们所表示的整数0、1、2。这些符号常量在unistd.h中定义。  

----------------------
## 缓冲IO和无缓冲IO
[带不带缓存是相对层来说的](https://www.cnblogs.com/cavehubiao/p/3981482.html)。  
如果你要写入数据到文件上时（就是写入磁盘上），内核先将数据写入到内核中所设的缓冲储存器。  
假如这个缓冲储存器的长度是100个字节，你调用系统函：   
```c
ssize_t write (int fd,const void * buf,size_t count);
```
写操作时，设每次写入长度count=10个字节，那么你几要调用10次这个函数才能把这个缓冲区写满，**此时数据还是在缓冲区，并没有写入到磁盘。**  
缓冲区满时才进行实际上的IO操作，把数据写入到磁盘上，所以上面说的“不带缓存""不是没有缓存而是没有直写进磁盘。

那么，既然**不带缓存的操作实际在内核是有缓存器的**，那**带缓存的IO操作**又是怎么回事呢？

带缓存IO也叫标准IO，符合ANSI C 的标准IO处理，不依赖系统内核，所以移植性强，我们使用标准IO操作很多时候是为了减少对read()和write()的系统调用次数，带缓存IO其实就是在用户层再建立一个缓存区，这个缓存区的分配和优化长度等细节都是标准IO库代你处理好了，不用去操心.  
还是用上面那个例子说明这个操作过程：  
内核缓存（注意这个不是用户层缓存区）区长度是100字节，  
我们调用不带缓存的IO函数write()就要调用10次，这样系统效率低，  
现在我们在用户层建立另一个缓存区（用户层缓存区或者叫流缓存），假设流缓存的长度是50字节，  
我们用标准C库函数的fwrite()将数据写入到这个流缓存区里面，流缓存区满50字节后在进入内核缓存区，此时再调用系统函数write()将数据写入到文件（实质是磁盘）上，  
标准IO操作fwrite()最后还是要掉用无缓存IO操作write,这里进行了两次调用fwrite()写100字节也就是进行两次系统调用write()。  

# lseek: 定位偏移指针
# truncate和ftruncate：截短文件
# umask： 在创建期间修改文件权限
# 目录导航--getcwd
# 读取目录

```c
//打开目录
DIR *opendir(const char *dirname);
//读打开的目录，这个结构体至少有目录的inode和dirName两个成员
struct dirent *readdir(DIR *dirp);
//关闭打开的目录
int closedir(DIR *dirp)
```

# 修改目录中的项目
1. mkdir和rmdir——这些调用的意义与同名命令相同
2. link和symlink——link创建一个硬链接，symlink创建一个符号链接
3. unlink——和rm差不多，删除一个普通文件或符号链接，它不会删除目录
4. rename——和mv一样

# 读取Inode：struct stat 和 stat
```c
struct stat{
  ino_t st_ino ; //Inode 编号
  mode_t st_mode; //模式(类型和权限)
  nlink_t st_nlink; //硬链接数

  uid_t st_uid; //UID(所有者)
  gid_t st_gid; //GID(用户组所有者)

  dev_t st_rdev; //设备ID(用于设备文件)
  off_t st_size; //文件大小(字节)

  time_t st_atime; //最后访问时间
  time_t st_mtime; //最后修改时间
  time_t st_ctime; //inode 最后修改时间

  blksize_t st_blksize ; //IO的优选块大小
  blkcnt_t st_blocks; //分配的块数
};
```
stat系统调用：  
```c
int stat(const char *path, struct stat *buf);
int lstat(const char *path, struct stat *buf);

int fstat(int fildes, struct stat *buf);
```
stat和lstat的第一参数需要路径名，第二参数是一个stat结构的地址（把第一参数下的文件信息以stat结构存放在第二参数中）。stat和lstat的区别是，对于符号链接，stat会沿着符号链接前进：如果，文件foo是链接到bar的(foo->bar)，那么对foo进行stat操作，**stat会用bar的属性填充stat结构**, 而 **lstat会提取foo的属性**。其他方面，stat和lstat没有区别。lstat更常用。  
fstat的第一参数是文件说明符，第二参数同样是stat结构地址。  

# access：检查实际用户的权限
```c
int access(const char *path, int amode);
```
第一参数是 **路径名**, 第二参数指定了 **要测试的访问权限** 。  
amode可以取以下4个值：  
R_OK ——读取权限正常  
W_OK ——写权限OK
X_OK ——执行权限OK
F_OK ——文件存在

# 修改文件属性
## chmod和fchmod：改变文件权限
## chown：改变所有权
## utime：改变时间戳
