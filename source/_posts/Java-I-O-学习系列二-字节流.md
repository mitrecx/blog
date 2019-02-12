---
title: 'Java I/O 学习系列二: 字节流'
date: 2019-01-23 08:55:17
tags: java
categories: java
---

**I/O流** 是 (输入/输出)读取/写入 "文件" 内容的技术。  

----

# 1 Java I/O 流简介

Java 程序 **通过 I/O流完成 输入/输出**。   
流是生产和消费信息的抽象。  
输入流可以是多种不同类型的输入：磁盘文件，键盘，网络；   
输出流可以是多种不同类型的输出：磁盘文件，控制台，网络。  

Java I/O流 的分类：  
根据 **流向** 分为 **<font color='#1a75ff'>输入流</font>** 和 **<font color='#1a75ff'>输出流</font>**；  
根据 **传输数据单位** 分为 **<font color='#1a75ff'>字节流</font>** 和 **<font color='#1a75ff'>字符流</font>**；  
根据 **功能** 分为 **<font color='#1a75ff'>节点流</font>** 和 **<font color='#1a75ff'>包装流</font>** (见标题4)。


**字节流**： Java 所有字节流都直接或间接 继承 InputStream类 和 OutputStream类。  
**字符流**： Java 所有字符流都直接或间接 继承 Reader类 和 Writer类。  

# 2 InputStream 和 OutputStream

InputStream类 是抽象类， OutputStream类 也是抽象类。  

## 2.1 InputStream
This abstract class is the superclass of all classes representing an input stream of bytes.  
**这个抽象类是 所有字节输入流的 父类**。  

### InputStream 主要方法：  
**<font color='#2eb82e'>abstract int	read()</font>**: **Reads the next byte** of data from the input stream. **返回 下一个字节内容(即本次read读取到的字节内容)，返回-1表示一个字节都没有读到(这意味着已经读到文件尾后)**。   
**<font color='#2eb82e'>int	read(byte[] b)</font>**: 等价于 read(byte[] b, 0, b.length)。**返回 实际读取到的字节个数，返回-1表示一个字节都没有读到(这意味着已经读到文件尾后)**。读取到的内容放在 byte数组中。    
**<font color='#2eb82e'>int	read(byte[] b, int off, int len)</font>**: 从输入流中，从0位置开始 读取 len个字节放入 b数组中(注：从off位置开始放)。**返回 实际读取到的字节个数，返回-1表示一个字节都没有读到(这意味着已经读到文件尾后)**。    
**<font color='#ff9900'>void	mark(int readlimit)</font>**: **Marks the current position** in this input stream.   
**<font color='#ff9900'>void	reset()</font>**: Repositions this stream to the position at the time the mark method was last called on this input stream.  
**<font color='#ff9900'>long	skip(long n)</font>**: **Skips over and discards n bytes** of data from this input stream.  
void	close(): Closes this input stream and releases any system resources associated with the stream.  

### InputStream 同包下的直接子类：  
ByteArrayInputStream  
**<font color='#1a75ff'>FileInputStream</font>**  
**<font color='#1a75ff'>FilterInputStream</font>**  
ObjectInputStream  
PipedInputStream  
SequenceInputStream    
StringBufferInputStream    


## 2.2 OutputStream
This abstract class is the superclass of all classes representing an output stream of bytes. An output stream accepts output bytes and sends them to some sink.  
**这个抽象类是 所有字节输出流的 父类**。输出流 接收输出字节 并发送到某个接收位置。  

### OutputStream 所有方法：  
**<font color='#2eb82e'>void	write(byte[] b)</font>**: 从 byte数组 取 b.length 个字节 写入输出流。等价于write(byte[] b, 0, b.length)。   
**<font color='#2eb82e'>void	write(byte[] b, int off, int len)</font>**: **Writes len bytes** from the specified byte array starting at offset off **to this output stream**. **off 是 在数组b 中的任意一个下标，len表示取 len个字节，off和len必须满足关系：len<=b.length-off**，否则会数组下标越界异常。   
**<font color='#2eb82e'>abstract void	write(int b)</font>**: 写 一个字节数据(b对应的ASCII字符) 到输出流，通常和 int read() 联合使用。  
**<font color='#ff9900'>void	flush()</font>**: Flushes this output stream and forces any buffered output bytes to be written out.  
void	close(): 略。  

### OutputStream 同包下的直接子类：  
ByteArrayOutputStream   
**<font color='#1a75ff'>FileOutputStream</font>**   
**<font color='#1a75ff'>FilterOutputStream</font>**   
ObjectOutputStream  
PipedOutputStream  


# 3 节点流: FileInputStream 和 FileOutputStream

## 3.1 FileInputStream

### 简介：  
A **FileInputStream** obtains **input bytes** from a file in a file system. What files are available depends on the host environment.  
**FileInputStream** is meant for reading streams of raw bytes such as image data. For reading streams of characters, consider using FileReader.  
**FileInputStream类** 用于从文件系统的文件中获取 **输入字节流**，文件是否可用取决于 主机环境。  
**FileInputStream类** 常用于读取文件的(原始)字节，如图片文件，视频文件等。如果读取字符文件(字符流)，应该用FileReader，而不是FileInputStream。  

### constructors
**<font color='#8a00e6'>FileInputStream(File file)</font>** throws FileNotFoundException：  
Creates a FileInputStream by opening a connection to an actual file, the file named by the File object file in the file system. **传入File对象，建立文件输入流**。  
**<font color='#8a00e6'>FileInputStream(String name)</font>** throws FileNotFoundException：  
Creates a FileInputStream by opening a connection to an actual file, the file named by the path name in the file system. **不创建File对象，直接传入文件路径**。  
**<font color='#8a00e6'>FileInputStream(FileDescriptor fdObj)</font>**：  
Creates a FileInputStream by using the file descriptor fdObj, which represents an existing connection to an actual file in the file system.  

**FileInputStream类** 的主要方法 和 **InputStream类** 的基本相同。  

### FileInputStream 示例
```java
public static void main(String[] args){
    File f=new File("C:/Users/cx141/Documents/test/file1.txt");
    int length=10;//(int)f.length();
    byte[] b=new byte[length];
    int num=0;
    try {
        FileInputStream fis=new FileInputStream(f);
        while((num=fis.read(b))!=-1){
            System.out.println("此次读取到的字节个数: "+num);
            for(int i=0;i<num;i++)
                System.out.print(b[i]);
            System.out.println();
            String str=new String(b,0,num);
            System.out.println("此次读取到的内容: "+str);
            System.out.println("-----");
        }
        fis.close();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }catch(IOException e){
        e.printStackTrace();
    }
}
```
file1.txt 文件内容如下：  
```
123456789
hello world
```
程序运行结果：  
```
此次读取到的字节个数: 10
49505152535455565713
此次读取到的内容: 123456789
-----
此次读取到的字节个数: 10
1010410110810811132119111114
此次读取到的内容:
hello wor
-----
此次读取到的字节个数: 2
108100
此次读取到的内容: ld
-----
```
注：  
由输出结果看，file1.txt 文件中 9后面的回车是占两个字节的。

## 3.2 FileOutputStream

### 简介：  
**A file output stream is an output stream for writing data to a File** or to a FileDescriptor. Whether or not a file is available or may be created depends upon the underlying platform. Some platforms, in particular, allow a file to be opened for writing by only one FileOutputStream (or other file-writing object) at a time. In such situations the constructors in this class will fail if the file involved is already open.  
**FileOutputStream is meant for writing streams of raw bytes such as image data**. For writing streams of characters, consider using FileWriter.  
**文件输出流 是把数据写入文件的输出流**。文件是否可用或文件是否可创建取决于底层平台。尤其在某些平台中，一次只能用一个 FileOutputStream流 进行写入。如果文件已经(用文件输出流)打开，那么再创建此文件的文件输出流会失败。  
**文件输出流 是用于写入(原始)字节到文件的流**，常用于写图片，视频等文件。 写入字符的流时应该使用FileWriter 而不是 FileOutputStream。  

### constructors
**<font color='#8a00e6'>public FileOutputStream(File file)</font>** throws FileNotFoundException:  
Creates a file output stream to write to **the file represented by the specified File object**.  

**<font color='#8a00e6'>FileOutputStream(File file, boolean append)</font>** throws FileNotFoundException:  
Creates a file output stream to write to **the file represented by the specified File object**. If the second argument is true, then bytes will be **written to the end of the file rather than the beginning**.  

**<font color='#8a00e6'>FileOutputStream(String name)</font>** throws FileNotFoundException:  
Creates a file output stream to write to **the file with the specified name**.  

**<font color='#8a00e6'>FileOutputStream(String name, boolean append)</font>** throws FileNotFoundException:  
Creates a file output stream to write to **the file with the specified name**. If the second argument is true, then bytes will be **written to the end of the file rather than the beginning**.   

**<font color='#8a00e6'>FileOutputStream(FileDescriptor fdObj)</font>**:  
Creates a file output stream to write to the specified file descriptor, which represents an existing connection to an actual file in the file system.   

**FileOutputStream类** 的主要方法 和 **OutputStream类** 的基本相同。  

### 示例
```java
public static void main(String[] args) {
    try {
        FileOutputStream fos = new FileOutputStream("C:/Users/cx141/Documents/test/file2.txt");
        String str="don't panic.";
        fos.write(str.getBytes());
        fos.write(10);// ASCII对应的字符是 换行符
        fos.write(49);// ASCII对应的字符是 1
        fos.write(97);// ASCII对应的字符是 a
        fos.write(10);
        fos.write("123456789".getBytes(),2,7); //注意：off 和 len 取值 受到byte[] b长度影响
        fos.close();
    }catch (FileNotFoundException e){
        e.printStackTrace();
    }catch (IOException e){
        e.printStackTrace();
    }
}
```

程序运行结果：  
```
don't panic.
1a
3456789
```

# 4 处理流: FilterInputStream 和 FilterOutputStream
**节点流**：可以从或向一个特定的地方(节点)读写数据。前面介绍的流 都是节点流。  
**处理流**：对一个已存在的流进行封装/装饰，通过对 封装流调用 实现数据读写。  

FilterInputStream 和 FilterOutputStream 都是 **处理流**。  

Filter这两个流用到了 设计模式中的 **装饰器模式**：  
FilterInputStream 和 FilterOutputStream 都是 **装饰器模式** 中的 Decorator抽象装饰角色。  
FilterInputStream 继承 InputStream，同时 FilterInputStream 有一个 InputStream 类型的成员。  
FilterOutputStream 继承 OutputStream，同时 FilterOutputStream 有一个 OutputStream 类型的成员。  

**<font color='#1a75ff'>FilterInputStream 和 FilterOutputStream  既然作为抽象的装饰器角色存在，所以他们没有什么实质性的方法</font>**。  

## 4.1 FilterInputStream

### FilterInputStream 简介
**A FilterInputStream contains some other input stream, which it uses as its basic source of data, possibly transforming the data along the way or providing additional functionality.**   
The class FilterInputStream itself simply overrides all methods of InputStream with versions that pass all requests to the contained input stream.   
Subclasses of FilterInputStream may further override some of these methods and may also provide additional methods and fields.  
**FilterInputStream 包含一些其他的输入流，FilterInputStream使用这些输入流作为基本数据源，并为它们提供额外功能**。  
FilterInputStream类 本身只是简单覆盖 InputStream 中的所有方法 (with versions这段不理解啥意思)。  
FilterInputStream 的子类可能进一步覆盖一些方法，并且可能添加了额外的方法和字段。  

### FilterInputStream 同包下的直接子类
BufferedInputStream  
DataInputStream  
PushbackInputStream  
LineNumberInputStream  

## 4.2 FilterOutputStream
### FilterOutputStream 简介
**This class is the superclass of all classes that filter output streams**.   
These streams sit on top of an already existing output stream (the underlying output stream) which it uses as its basic sink of data, but possibly transforming the data along the way or providing additional functionality.  
The class FilterOutputStream itself simply overrides all methods of OutputStream with versions that pass all requests to the underlying output stream. Subclasses of FilterOutputStream may further override some of these methods as well as provide additional methods and fields.  
**FilterOutputStream类 是所有过滤输出流的超类**。  
这些 filter output stream 的作用是 包装其他的流 并提供额外的功能。  

### FilterOutputStream 同包下的直接子类
BufferedOutputStream  
DataOutputStream  
PrintStream  

# 5 处理流: BufferedInputStream 和 BufferedOutputStream
继承关系：  
```
InputStream
    |__FilterInputStream
            |__BufferedInputStream

OutputStream
    |__FilterOutputStream
            |__BufferedOutputStream
```
缓冲输入/输出流 的思想：  
缓冲输入/输出流 每次 读取/写入 数据时，先把数据放在 **缓冲区**(内存)中，等缓冲区满了(或 **调用flush函数**)再进行 **实际的读写操作**。  

缓冲输入/输出流 读写 **效率** 比其他流高，比如：    
不带缓冲的输出流，写入文件，每次读几个字节就写几个字节，I/O次数比较多；  
缓冲输出流，写入文件，可以一次读很多字节，但不向磁盘中写入，只是先放到内存里，等凑够了缓冲区大小的时候一次性写入磁盘，这样可以减少磁盘操作次数，提高写入效率。    

## 5.1 BufferedInputStream
BufferedInputStream 为 另一个输入流添加了 mark 和 reset 的功能。  
BufferedInputStream 对象创建时，同时会创建一个 **内部缓冲区数组**(internal buffer array)。  
当 read 或 skip 流中的字节时，内部缓冲区会被重新填充。  
mark 操作会标记输入流中的一个位置，reset 操作 会跳转到最近的 mark 的位置。  


### Fields
```java
// 内部缓冲区数组
protected byte[] buf;

// buf 中存储的字节 总数
protected int count;

// 缓冲区当前位置(下标)
protected int pos;

// 上次调用 mark 时 记录的 pos的值
protected int	markpos;

// 不理解这个字段的意思
protected int	marklimit;

// 从 java.io.FilterInputStream 继承的字段
protected volatile InputStream in;
```
### Constructors
```java
// 使用默认buf大小, 8192字节(8KB)
public BufferedInputStream(InputStream in);

// 使用 size 指定的 buf 大小
public BufferedInputStream(InputStream in,int size);
```

### Methods
```java
// 返回 被包装流 的可读取的字节总数 的估计值
int	available();

// 和 InputStream 中的 read 一样：读取下一个字节
int	read();
// 给一个起始位置 off，从字节输入流读取 len个字节 放入b数组中
int	read(byte[] b, int off, int len);

// 查看此流是否支持 mark
boolean markSupported();
// 同 InputStream：标记当前 位置
void mark(int readlimit);
// 同 InputStream
void reset();
// 同 InputStream
long skip(long n);

void close();
```

## 5.2 BufferedOutputStream
BufferedOutputStream 包装 **底层输出流(underlying output stream)** 提供缓冲功能。  

### Fields
```java
// 内部缓冲区数组
protected byte[] buf;

// The number of valid bytes in the buffer. 缓冲区数组的有效字节数(即，数组实际大小)
protected int count;

// 继承 java.io.FilterOutputStream
protected OutputStream out;
```

### Constructors
```java
// 使用 默认 内部缓冲数组大小 8192字节(8KB)
public BufferedOutputStream(OutputStream out);
// 使用 size 指定 内部缓冲数组大小
public BufferedOutputStream(OutputStream out, int size);
```

### Methods
```java
void flush();
// 从 b数组中 取 len 个字节 写入 缓冲输入流
void write(byte[] b, int off, int len);
// 写指定的字节到 缓冲输出流
void write(int b);

// 还有一些继承来的方法，略
```

## 5.3 缓冲输入/输出流 示例

以下四段程序复制的 zip 文件 大小为 1.12GB。  

1、 **FileInputStream/FileOutputStream** 单字节复制：  
```java
public static void main(String[] args){
    long begin=System.currentTimeMillis();
    File fileIn=new File("C:\\Users\\cx141\\Downloads\\win64_11gR2_database_1of2.zip");
    File fileOut=new File("C:/Users/cx141/Documents/test/1.zip");
    try {
        FileInputStream fis = new FileInputStream(fileIn);
        FileOutputStream fos = new FileOutputStream(fileOut);
        int i=0;
        while((i=fis.read())!=-1){
            fos.write(i);
        }
        fos.close();
        fis.close();
        long end=System.currentTimeMillis();
        System.out.println("耗时: "+ (end-begin)/1000 +" 秒");
    }catch (FileNotFoundException e){
        e.printStackTrace();
    }catch (IOException e){
        e.printStackTrace();
    }
}
//耗时大约 1 个小时
```

2、 **<font color=red>BufferedInputStream</font>/<font color=red>BufferedOutputStream</font>** 单字节复制：  
```java
public static void main(String[] args){
    long begin=System.currentTimeMillis();
    try{
        File f1=new File("C:/Users/cx141/Downloads/win64_11gR2_database_1of2.zip");
        FileInputStream in = new FileInputStream(f1);
        BufferedInputStream bin=new BufferedInputStream(in);
        File f2=new File("C:/Users/cx141/Documents/test","2.zip");
        FileOutputStream out = new FileOutputStream(f2);
        BufferedOutputStream bout=new BufferedOutputStream(out);
        int b=0;
        while((b=bin.read())!=-1){
            bout.write(b);
        }
        bout.flush();
        bin.close();
        bout.close();
        long end=System.currentTimeMillis();
        long consume=(end-begin)/1000;
        System.out.println("耗时："+consume+" 秒");
    }catch(FileNotFoundException e){
        e.printStackTrace();
    }catch(IOException e){
        e.printStackTrace();
    }
}
// 耗时：40 秒
```
**单字节复制时，结果很明显，缓冲流包装文件流复制 比 直接用文件流复制 要快得多**。  

3、 **FileInputStream/FileOutputStream** 多字节(2KB)复制：  
```java
public static void main(String[] args){
    long begin=System.currentTimeMillis();
    File fileIn=new File("C:\\Users\\cx141\\Downloads\\win64_11gR2_database_1of2.zip");
    File fileOut=new File("C:/Users/cx141/Documents/test/3.zip");
    try {
        FileInputStream fis = new FileInputStream(fileIn);
        FileOutputStream fos = new FileOutputStream(fileOut);
        int BUFSIZE=1024*2;
        byte[] b=new byte[BUFSIZE];
        int readLength=0;
        while((readLength=fis.read(b,0,BUFSIZE))!=-1){
            fos.write(b,0,BUFSIZE);
        }
        fos.close();
        fis.close();
        long end=System.currentTimeMillis();
        System.out.println("耗时: "+ (end-begin)/1000 +" 秒");
    }catch (FileNotFoundException e){
        e.printStackTrace();
    }catch (IOException e){
        e.printStackTrace();
    }
}
// 耗时：6 秒
```

4、 **<font color=red>BufferedInputStream</font>/<font color=red>BufferedOutputStream</font>** 多字节(2KB)复制：  
```java
public static void main(String[] args){
    long begin=System.currentTimeMillis();
    File fileIn=new File("C:\\Users\\cx141\\Downloads\\win64_11gR2_database_1of2.zip");
    File fileOut=new File("C:/Users/cx141/Documents/test/4.zip");
    try {
        FileInputStream fis=new FileInputStream(fileIn);
        FileOutputStream fos=new FileOutputStream(fileOut);
        // 缓冲输入/输出流
        BufferedInputStream bis=new BufferedInputStream(fis);
        BufferedOutputStream bos=new BufferedOutputStream(fos);
        int BUFSIZE=1024*2;
        byte[] buf=new byte[BUFSIZE];
        int readLength=0;
        while((readLength=bis.read(buf,0,BUFSIZE))!=-1){
            bos.write(buf,0,readLength);
        }
        bos.flush();
        bis.close();
        bos.close();
        long end=System.currentTimeMillis();
        System.out.println("耗时: "+ (end-begin)/1000 +" 秒");
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e){
        e.printStackTrace();
    }
}
//耗时：2 秒
```
总结与分析：  
上面程序中，缓冲流均采用 默认的 8KB 大的 缓冲byte数组。  
不论是单字节还是多字节复制，利用 **缓冲流** 处理的文件流 都比 普通文件流 要快(因为缓冲到内存，减少了 I/O 次数)。  
如果文件不大，采用多字节复制，普通文件流操作 和 缓冲流 之间的效率差距并不大！  
**<font color=red>缓冲流只有在大幅减少 I/O 次数时，才可能提高效率</font>**。  

# 参考
1、 [深入理解JAVA I/O系列二：字节流详解](https://www.cnblogs.com/dongguacai/p/5658388.html)  
2、 [Standard Edition 8 API Specification](https://docs.oracle.com/javase/8/docs/api/)  
3、 [Java IO详解（二)------流的分类](https://www.cnblogs.com/ysocean/p/6854098.html)  
4、 [JavaIO之FilterInputStream FilterOutputStream](https://www.cnblogs.com/noteless/archive/2018/09/12/9632812.html)  
5、 [Java IO流学习总结三：缓冲流](https://blog.csdn.net/zhaoyanjun6/article/details/54894451)  
