---
title: 'Java I/O 学习系列一: File类'
date: 2019-01-22 11:41:37
tags: java
categories: java
---
抽象路径名(abstract pathname)：Java 为了方便对不同命名规则的路径进行统一的处理，以 Java的抽象路径名代表不同系统的路径名。   

----

File 类 简介：  
1、一个 File类 的对象，表示磁盘上的一个文件/目录。  
2、File 提供了 (与平台无关的)方法来对磁盘上的 文件/目录 进行操作。  

**可见，File类 可以表示文件及操作文件， 但 <font color=red>不能读写文件内容</font>。**

# 1 Constructors
File 类的 有 4个 constructors：  
```java
File(File parent, String child)
// 根据 文件或目录 创建File对象
File(String pathname)

File(String parent, String child)

File(URI uri)
```
注：  
File 对象 对应的文件/目录 不一定存在。  

# 2 常用 [Methods](https://docs.oracle.com/javase/8/docs/api/)

**<font color='#1a75ff'>boolean	canRead()</font>**  文件/目录 是否可读  
**<font color='#1a75ff'>boolean	canWrite()</font>**   
**<font color='#1a75ff'>boolean	canExecute()</font>**  

**<font color='#1a75ff'>boolean	exists()</font>** 判断 文件/目录 是否存在，存在返回 true   
**<font color='#1a75ff'>boolean	createNewFile()</font>**  创建 文件/目录   
**<font color='#1a75ff'>boolean	delete()</font>**  删除 文件/目录  

**<font color='#1a75ff'>String	getAbsolutePath()</font>** 获取抽象路径名(文件/目录)的绝对路径(文件或目录)  
**<font color='#1a75ff'>String	getPath()</font>**  获取抽象路径名(文件/目录)的路径(文件或目录)   
**<font color='#1a75ff'>String	getName()</font>**  获取文件/目录名  

**<font color='#1a75ff'>boolean	isDirectory()</font>**  是目录返回true   
**<font color='#1a75ff'>boolean	isFile()</font>**  是文件返回true  

**<font color='#1a75ff'>long	length()</font>**   返回文件的大小(字节)   
**<font color='#1a75ff'>File[]	listFiles()</font>**  返回目录下的所有 文件/目录  

**<font color='#1a75ff'>boolean	mkdir()</font>**  创建目录  
**<font color='#1a75ff'>boolean	mkdirs()</font>**  递归创建多次目录  

# 3 示例

```java
public static void main(String[] args) {
    File f1=new File("C:\\Users\\cx141\\Documents\\test\\file1.txt");
    // 指定路径的文件是否存在，存在 exists 返回true
    if(f1.exists()) {
        System.out.println("文件名称: "+f1.getName());
        System.out.println("文件大小: "+f1.length()+" 字节");
        System.out.println("文件绝对路径: "+f1.getAbsolutePath());
    }else {
        System.out.println("file1.txt 不存在");
        try {
            //createNewFile 创建文件
            f1.createNewFile();
            System.out.println("file1.txt 文件创建成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 删除文件
    File f2 =new File("C:/Users/cx141/Documents/test/file1.txt");
    f2.delete();

    // 创建目录
    File f3=new File("C:/Users/cx141/Documents/test/dirc2");
    if(f3.exists() && f3.isDirectory()) {
        System.out.println("dirc2 目录已存在");
    }else {
        f3.mkdir(); //mkdir创建目录， 另：mkdirs 可以递归创建多层目录
        System.out.println("dirc2 目录创建成功");
    }

    // listFiles 列出目录中所有的文件和文件夹
    File f4=new File("C:/Users/cx141/Documents/test");
    if(f4.isDirectory()) {
        File fList[]=f4.listFiles();
        for(int i=0;i<fList.length;i++) {
            System.out.println(fList[i]);
        }
    }
}
```
在 Windows OS 中，绝对路径写法：<code>C:\\Users\\cx141\\Documents\\test\\file1.txt</code> ，  
也可以 <code>C:/Users/cx141/Documents/test/file1.txt</code> 。  
在 Linux/Unix 中， 绝对路径写法：<code>C:/Users/cx141/Documents/test/file1.txt</code> 。  
所以，**路径可以统一写作**：<code>C:/Users/cx141/Documents/test/file1.txt</code> 。   

注：  
在File类中，路径的分隔符是 <code>File.separator</code>   


# 参考  
1、 [JAVA 抽象路径](https://blog.csdn.net/crslee/article/details/51043045?utm_source=blogxgwz2)  
2、 [深入理解JAVA I/O系列一：File](https://www.cnblogs.com/dongguacai/p/5656471.html)  
