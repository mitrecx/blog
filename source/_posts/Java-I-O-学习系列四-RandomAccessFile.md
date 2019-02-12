---
title: "Java I/O 学习系列四: RandomAccessFile"
date: 2019-02-11 16:08:14
tags: java
categories: java
---
# 1 RandomAccessFile 简介

## 1.1 API docs 对 RandomAccessFile 的描述
```
java.lang.Object
      |__java.io.RandomAccessFile
```
Instances of this class support both reading and writing to a random access file.   
A random access file behaves like a large array of bytes stored in the file system.   
There is a kind of cursor, or index into the implied array, called **<font color='#ff9900'>the file pointer</font>**;   
**input operations** read bytes starting at the file pointer and advance the file pointer past the bytes read.   

If the random access file is created in read/write mode, then **output operations** are also available;   
output operations write bytes starting at the file pointer and advance the file pointer past the bytes written.   
Output operations that write past the current end of the implied array cause the array to be extended.   
The file pointer can be read by the **<font color='#8a00e6'>getFilePointer</font>** method and set by the **<font color='#8a00e6'>seek</font>** method.  

It is generally true of all the reading routines in this class that if **end-of-file** is reached before the desired number of bytes has been read, an **<font color='#ff0000'>EOFException</font>** (which is a kind of IOException) is thrown.   
If any byte cannot be read for any reason other than end-of-file, an IOException other than(除了) EOFException is thrown.   
In particular, an **<font color='#ff0000'>IOException** may be thrown if the stream has been closed.  
RandomAccessFile类 在读取程序中，如果在读取完 所需的字节数 之前到达 文件结尾，将会抛出 EOFException 。  
除 读到文件结尾 之外的任何原因，导致不能读取到 字节，将会抛出 IOException。  

## 1.2 RandomAccessFile 的作用
RandomAccessFile 类 实现了 DataInput 和 DataOutput 接口，所以 RandomAccessFile类 既可以读也可以写。  

RandomAccessFile 可以 **<font color='#ff0000'>访问文件</font>** 的任意位置，并且支持 读取和写入 操作。  

## 1.3 构造函数
```java
public RandomAccessFile(String name, String mode) throws FileNotFoundException
public RandomAccessFile(File file, String mode) throws FileNotFoundException
```
mode 参数用于指定访问模式：  
"r" 只读  
"rw" 读写  
"rws" 读写，同时对文件的内容或 元数据(metadata) 的更新 同步写入底层存储设备  
"rwd" 读写，同时对文件的内容的更新 同步写入底层存储设备

# 2 RandomAccessFile 用法示例

## 2.1 写入和读取文件
**<font color='#8a00e6'>writeUTF</font>** 方法 使用 modified UTF-8 encoding 以独立于机器的方式把字符串写入文件。  

用 **<font color='#8a00e6'>writeUTF</font>** 方法 把中文字符串写入文件中，直接打开文件看到的是 **<font color='#ff0000'>乱码</font>**。  
使用 **指定的write 方法**(如，writeUTF，writeInt，writeLong等) 写入，应该使用 **对应的read 方法**(如，readUTF，readInt，readLong等)读取。  

例如：  
```java
public static void main(String[] args) {
  //写入
  write();
  try {
    RandomAccessFile raf=new RandomAccessFile("C:\\Users\\cx141\\Desktop\\credit\\1.txt", "rw");
    String s=raf.readUTF();
    String s2=raf.readUTF();
    int s3=raf.readInt();
    System.out.println(s+s2+s3);
    raf.close();
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  }catch (IOException e) {
    e.printStackTrace();
  }
}

public static void write() {
  try {
    RandomAccessFile raf = new RandomAccessFile("C:\\Users\\cx141\\Desktop\\credit\\1.txt", "rw");
    raf.writeUTF("hello world.");
    raf.writeUTF("你好 ");
    raf.writeInt(123);
    raf.close();
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```
执行结果：  
```
hello world.你好 123
```

## 2.2 在文件结尾插入

```java
public static void main(String[] args) {
  try {
    RandomAccessFile raf = new RandomAccessFile("C:\\Users\\cx141\\Desktop\\credit\\2.txt", "rw");
    raf.seek(raf.length());
    raf.write("\r\n江阴农商银行".getBytes());
    raf.close();
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```
程序执行后 文件2.txt 内容如下(不 **<font color='#ff0000'>乱码</font>**)：  
```
中华人民共和国
美利坚合众国
江阴农商银行
```

注：读取文件 可以按行读  
```java
RandomAccessFile raf = new RandomAccessFile("C:\\Users\\cx141\\Desktop\\credit\\2.txt", "rw");
String line = raf.readLine();
while (line != null) {
  System.out.println(new String(line.getBytes("ISO-8859-1"), "GBK"));
  line = raf.readLine();
}
```


## 2.3 在文件的任意位置插入
RandomAccessFile 在 任意位置pos插入 会覆盖pos后面原有的数据。  
只要把pos后面的内容先保存在一个临时文件中，待插入完成后，再把临时文件中的内容添加到 原文件结尾即可。  

例如：  
```java
public static void main(String[] args) {
  // 指定一个插入位置
  int pos = 14;
  File temp = new File("C:\\Users\\cx141\\Desktop\\credit\\temp.txt");
  try {
    temp.createNewFile();
    RandomAccessFile raf = new RandomAccessFile("C:\\Users\\cx141\\Desktop\\credit\\2.txt", "rw");
    FileOutputStream fos = new FileOutputStream(temp);
    raf.seek(pos);
    byte[] buffer = new byte[2];
    int num = 0;
    // 把 seek 后的数据全部存储到临时文件中
    while (-1 != (num = raf.read(buffer))) {
      fos.write(buffer, 0, num);
    }
    fos.close();
    raf.seek(pos);
    raf.write("今天成立了".getBytes());
    FileInputStream fis = new FileInputStream(temp);
    // 把临时文件内容 添加到原文件结尾
    while (-1 != (num = fis.read(buffer))) {
      raf.write(buffer, 0, num);
    }
    fis.close();
    raf.close();
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```
程序执行后，2.txt 文件内容：  
```
中华人民共和国今天成立了
美利坚合众国
江阴农商银行
```
