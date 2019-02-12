---
title: 'Java I/O 学习系列三: 字符流'
date: 2019-01-24 15:29:34
tags: java
categories: java
---

# 1 为什么需要字符流

纯英文环境下，只要 **字节流** 就能完成一切任务。  

在中文环境下，**字节流** 也能完成大多数 I/O 操作。  
但是遇到 读取中文文本文件，或 写入中文到文件中 这种操作时，就需要 **<font color='#2eb82e'>字符流</font>** 了。

注意：    
复制中文文本文件 **字节流** 可以做。  
但是 读取中文文本文件 并 打印到控制台 这种操作 **字节流** 做不了，
因为不知道中文字符占 2个字节(GBK) 还是 3个字节(UTF-8)，所以无法准确获取中文字符。  
这种情况下，**<font color='#2eb82e'>字符流</font>** 就能 准确地获取到 中文字符。因为 **<font color='#2eb82e'>字符流</font>** 是 按字符读取文件的。

# 2 Reader 和 Writer

Reader 是一个 **抽象类**， Writer 也是一个 **抽象类** 。  

## 2.1 Reader
### Reader 简介
**Abstract class for reading character streams**.   
The only methods that a subclass must implement are read(char[], int, int) and close().   
Most subclasses, however, will override some of the methods defined here in order to provide higher efficiency, additional functionality, or both.   
**Reader 是一个 读取字符流 的抽象类**。  
Reader 的子类 必须实现 read(char[], int, int) 和 close() 方法。  
然而，大多数子类 为了提高效率或附加功能 会覆盖许多其他的方法。  

### Fields
```java
// 用于同步流的操作。为提高效率，Reader的子类应该用此字段 而不是 this对象 或 synchronized方法
protected Object lock;
```
### Methods
```java
// Closes the stream and releases any system resources associated with it.
public abstract void close() throws IOException

// Marks the present position in the stream.
public void mark(int readAheadLimit) throws IOException

// Tells whether this stream supports the mark() operation.
public boolean markSupported()

// Reads a single character.
public int read() throws IOException

// Reads characters into an array. 读取多个字符放在 cbuf 数组里
public int read(char[] cbuf) throws IOException

// Reads characters into a portion of an array.
public abstract int read(char[] cbuf, int off,int len) throws IOException

// Attempts to read characters into the specified character buffer.
public int read(CharBuffer target) throws IOException

// Tells whether this stream is ready to be read.
public boolean ready() throws IOException

// Resets the stream.
public void reset() throws IOException

// Skips n个 characters.
public long skip(long n) throws IOException
```
### Reader 的直接子类
**BufferedReader**   
CharArrayReader   
FilterReader   
**InputStreamReader**   
PipedReader   
StringReader  

## 2.2 Writer
**Abstract class for writing to character streams**.   
The only methods that a subclass must implement are write(char[], int, int), flush(), and close().   
Most subclasses, however, will override some of the methods defined here in order to provide higher efficiency, additional functionality, or both.   
**Writer 是一个写入字符流的 抽象类**。  
Writer 的子类必须要实现(覆盖) write(char[], int, int), flush(), 和 close() 方法。  
然而，大多数子类 为了提高效率或附加功能 会覆盖许多其他的方法。  

### Fields
```java
// 用于同步流的操作。 为提高效率，Writer的子类应该用此字段 而不是 this对象 或 synchronized方法
protected Object lock;
```

### Methods
```java
// Appends the specified character to this writer.
public Writer append(char c) throws IOException

// Appends the specified character sequence to this writer.
public Writer append(CharSequence csq) throws IOException

// Appends a subsequence of the specified character sequence to this writer.
public Writer append(CharSequence csq, int start, int end) throws IOException

// Closes the stream, flushing it first.
public abstract void close() throws IOException

// Flushes the stream.
public abstract void flush() throws IOException

// Writes an array of characters.
public void write(char[] cbuf) throws IOException

// Writes a portion of an array of characters. 写入字符数组的一部分。
public abstract void write(char[] cbuf, int off, int len) throws IOException

// Writes a single character.
public void write(int c) throws IOException

// Writes a string.
public void write(String str) throws IOException

// Writes a portion of a string. 写入字符串的一部分。
public void write(String str, int off, int len) throws IOException
```

### Writer 的直接子类
**BufferedWriter**   
CharArrayWriter   
FilterWriter  
**OutputStreamWriter**  
PipedWriter  
PrintWriter  
StringWriter  

# 3 InputStreamReader 和 OutputStreamWriter
继承关系：  
```
Reader
   |__InputStreamReader

Writer
   |__OutputStreamWriter
```

## 3.1 InputStreamReader
public class InputStreamReader extends Reader  

### InputStreamReader 简介
**An InputStreamReader is a bridge from byte streams to character streams**: It reads bytes and decodes them into characters using a specified charset.   
The charset that it uses may be specified by name or may be given explicitly, or the platform's default charset may be accepted.  
**InputStreamReader 是 字节流 到 字符流 的桥梁：** InputStreamReader 读取字节 并 使用指定的字符集 解码为字符。  
InputStreamReader 使用的字符集 可以通过名称指定，也可以显式给定，或者接受平台默认的字符集。  

每次调用 InputStreamReader 的 read 方法 可能导致从 **底层字节输入流(underlying byte-input stream)** 读取一个或多个字节。  
为了能有效的将字节转为字符，从 基础流中 提前读取的字节数 可能超过 当前读取操作所需的字节数。  
为了获得最高效率，一般在 BufferedReader 中 包装(wrapping) 一个 InputStreamReader ，例如：  
```java
BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
```
### Constructors
```java
// Creates an InputStreamReader that uses the default charset.
InputStreamReader(InputStream in)

// Creates an InputStreamReader that uses the given charset.
InputStreamReader(InputStream in, Charset cs)

// Creates an InputStreamReader that uses the given charset decoder.
InputStreamReader(InputStream in, CharsetDecoder dec)

// Creates an InputStreamReader that uses the named charset.
InputStreamReader(InputStream in, String charsetName)
```

## 3.2 OutputStreamWriter
public class OutputStreamWriter extends Writer

### OutputStreamWriter 简介
**An OutputStreamWriter is a bridge from character streams to byte streams**: Characters written to it are encoded into bytes using a specified charset.   
The charset that it uses may be specified by name or may be given explicitly, or the platform's default charset may be accepted.   
**OutputStreamWriter 是字符流 到 字节流 的桥梁**： 使用指定的字符集编码字符 后写入。  
OutputStreamWriter 使用的字符集 可以通过名称指定，也可以显式给定，或者接受平台默认的字符集。  

每次调用 OutputStreamWriter 的 write 方法都会 对待处理的字符调用 编码转换器(encoding converter)。  
字节被写入 **底层输入层流(underlying output stream)** 之前被累计放在 **缓冲区** 中。  
缓冲区 的大小可以指定，但通常默认的大小足以满足大多数情况。  
注意，传递给 write 方法的 字符不会被缓冲。  

为了获得最高效率，一般在 BufferedWriter 中 包装(wrapping) 一个 OutputStreamWriter ，这样可以避免频繁地调用转换器(converter)，例如：  
```java
Writer out = new BufferedWriter(new OutputStreamWriter(System.out));
```

### Constructors
```java
// Creates an OutputStreamWriter that uses the default character encoding.
OutputStreamWriter(OutputStream out)

// Creates an OutputStreamWriter that uses the given charset.
OutputStreamWriter(OutputStream out, Charset cs)

// Creates an OutputStreamWriter that uses the given charset encoder.
OutputStreamWriter(OutputStream out, CharsetEncoder enc)

// Creates an OutputStreamWriter that uses the named charset.
OutputStreamWriter(OutputStream out, String charsetName)
```

# 4 FileReader 和 FileWriter
继承关系：  
```
Reader
   |__InputStreamReader
               |__FileReader

Writer
   |__OutputStreamWriter
               |__FileWriter
```
## 4.1 FileReader
### FileReader 简介

**Convenience class for reading character files**.   
The constructors of this class assume that the default character encoding and the default byte-buffer size are appropriate.   
To specify these values yourself, construct an InputStreamReader on a FileInputStream.  
**FileReader 是一个 从文件读取字符的 便利类**。  
FileWriter 的构造器 全都假定 默认的字符编码和默认的字节缓冲大小 是 符合要求的( 可接受的 )。  
如果要自己指定这些值得话，需要在 FileInputStream 上构造 InputStreamReader。  

FileReader用于读取字符流。对于读取原始字节流，要使用 FileInputStream。

### Constructors
和 **(字节)文件输入流 FileInputStream** 一样，有 3 个构造函数，参数分别是 File、FileDescriptor、String：  
```java
// Creates a new FileReader, given the File to read from.
FileReader(File file)

// Creates a new FileReader, given the FileDescriptor to read from.
FileReader(FileDescriptor fd)

// Creates a new FileReader, given the name of the file to read from.
FileReader(String fileName)
```

## 4.2 FileWriter
### FileWriter 简介
**Convenience class for writing character files**.   
The constructors of this class assume that the default character encoding and the default byte-buffer size are acceptable.   
To specify these values yourself, construct an OutputStreamWriter on a FileOutputStream.
**FileWriter 是一个 写入字符到文件的 便利类**。  
FileWriter 的构造器 全都假定 默认的字符编码和默认的字节缓冲大小 是 符合要求的( 可接受的 )。  
如果要自己指定这些值得话，需要在 FileOutputStream 上构造 OutputStreamWriter。  

### Constructors
和 **(字节)文件输出流 FileOutputStream** 一样，有 5 个构造函数：  
```java
// Constructs a FileWriter object given a File object.
FileWriter(File file)

// Constructs a FileWriter object given a File object.
FileWriter(File file, boolean append)

// Constructs a FileWriter object associated with a file descriptor.
FileWriter(FileDescriptor fd)

// Constructs a FileWriter object given a file name.
FileWriter(String fileName)

// Constructs a FileWriter object given a file name with a boolean indicating whether or not to append the data written.
FileWriter(String fileName, boolean append)
```

# 5 处理流：BufferedReader 和 BufferedWriter
继承关系：  
```
Reader
   |__BufferedReader

Writer
   |__BufferedWriter
```

## 5.1 BufferedReader
### BufferedReader 简介
**Reads text from a character-input stream, buffering characters so as to provide for the efficient reading of characters, arrays, and lines**.  
The buffer size may be specified, or the default size may be used. The default is large enough for most purposes.  
**BufferedReader 从 字符输入流 中读取文本，缓冲字符 以便有效地读取字符、数组和行** 。  
可以指定缓冲区大小，也可以使用默认的缓冲区，在大多数场合默认缓冲区是够用的。  

In general, **each read request made of a Reader causes a corresponding read request to be made of the underlying character or byte stream**.   
It is therefore advisable to wrap a BufferedReader around any Reader whose read() operations may be costly, such as FileReaders and InputStreamReaders.   
通常， **Reader 的 read 请求 会触发 相应的 底层的字符或字节流(underlying character or byte stream)的 read 请求**。  
因此，建议将 BufferedReader 包装在 read() 操作代价高昂的 Reader 上，例如 FileReader 和 InputStreamreader。  
例如:  
```java
BufferedReader in  = new BufferedReader(new FileReader("foo.in"));
```

### Constructors
BufferedReader 的 构造函数 和 BufferedInputStream 类似：  
```java
// Creates a buffering character-input stream that uses a default-sized input buffer.
BufferedReader(Reader in)

// Creates a buffering character-input stream that uses an input buffer of the specified size.
BufferedReader(Reader in, int sz)
```

### 部分 Methods
```java
// Reads a single character.
int	read()

// Reads characters into a portion of an array.
int	read(char[] cbuf, int off, int len)

// Reads a line of text.
String	readLine()
```

## 5.2 BufferedWriter
### BufferedWriter 简介
**Writes text to a character-output stream, buffering characters so as to provide for the efficient writing of single characters, arrays, and strings**.  
The buffer size may be specified, or the default size may be accepted. The default is large enough for most purposes.  
**BufferedWriter 把文本写入到 字符输出流，缓冲字符 以便有效地 写入单个字符、数组、字符串**。  

A newLine() method is provided, which uses the platform's own notion of line separator as defined by the system property line.separator.   
Not all platforms use the newline character ( **'\\n'** ) to terminate lines.   
Calling this method to terminate each output line is therefore preferred to writing a newline character directly.  
BufferedWriter 提供了 newLine() 方法，这个方法是 用平台自己的 行分隔符(line separator) 来换行 (注：行分隔符定义在系统属性 line.separator 中)。  
不是所有的 platform 都支持  **'\\n'** 换行。  
因此，调用 newLine() 方法换行 比 直接用 写换行符换行 更可取。  

In general, a Writer sends its output immediately to the underlying character or byte stream.   
Unless prompt output is required, it is advisable to wrap a BufferedWriter around any Writer whose write() operations may be costly, such as FileWriters and OutputStreamWriters.  
通常， Writer 会立即发送它的输出 到 底层字符或字节流(the underlying character or byte stream)。  
除非需要提示输出，否则 建议 使用 BufferedWriter 包装 write() 操作代价高昂的 Writer 上，例如 FileWriter 和 OutputStreamWriter。  
例如：  
```java
PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("foo.out")));
```

### Constructors
BufferedWriter 的 构造方法 和 BufferedOutputStream 类似：  
```java
// Creates a buffered character-output stream that uses a default-sized output buffer.
BufferedWriter(Writer out)

// Creates a new buffered character-output stream that uses an output buffer of the given size.
BufferedWriter(Writer out, int sz)
```

### 部分 Methods
```java
// Writes a line separator.
void	newLine()

// Writes a portion of an array of characters.
void	write(char[] cbuf, int off, int len)

// Writes a single character.
void	write(int c)

// Writes a portion of a String.
void	write(String s, int off, int len)
```

# 6 PrintWriter
```
java.lang.Object
      |__java.io.Writer
            |__java.io.PrintWriter
```

PrintWriter 是一种 **字符输出流**。  
PrintWriter 的构造函数 可以接收的参数类型：  
1、 file对象  
2、 字符串对象  
3、 字节输出流 OutputStream  
4、 字符输出流 Writer  



示例：  
```java
public class PrintWriterDemo {

  public static void main(String[] args) throws IOException {
    BufferedReader bufr = new BufferedReader(new InputStreamReader(System.in));
    // true表示自动刷新
    PrintWriter out = new PrintWriter(new FileWriter("aaaa.txt"), true);
    String line = null;
    while ((line = bufr.readLine()) != null) {
      out.println(line);
    }
  }

}
```

# 7 综合示例
待更新...

# 参考
1、 [Java Standard Edition 8 API Specification](https://docs.oracle.com/javase/8/docs/api/)  
