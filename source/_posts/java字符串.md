---
title: java字符串
date: 2018-11-13 14:10:56
tags: java
categories: java
---
java 的 String 类中的所有方法的用法 都在 [JavaSE 8 API 的 docs](https://docs.oracle.com/javase/8/docs/api/) 中做了详细的说明。  

这里挑出几个常用的方法介绍，以巩固之所学。  

先看一例：  

**标准期限**：两位年，两位月，两位日(取零)。表示 **整月数** 。其中月 逢12进1 。  
例如：  
00 01 00 表示 1 个月。  
00 11 00 表示 11 个月。再加一个月 就是 01 00 00，表示 12 个月，即 1 年。  


把任意指定的一个 **标准期限** 转为 整月数：  
```java
public class TestString {
	public static int nstd2std(String stdTerm) {
		if (stdTerm == null || stdTerm.length() != 6) {
			System.out.println("标准期限必须6位");
		} else {
			String yearBit = stdTerm.substring(0, 2);
			int month_y = Integer.parseInt(yearBit) * 12;
			String monthBit = stdTerm.substring(2, 4);
			int month_m = Integer.parseInt(monthBit);
			int totleMonth = month_y + month_m;
			return totleMonth;
		}
		return 0;
	}

	public static void main(String[] args) {
		while (true) {
			System.out.print("输入: ");
			Scanner scan = new Scanner(System.in);
			String stdTerm = scan.nextLine();
			int term = nstd2std(stdTerm);
			System.out.println(term);
		}
	}

}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/javaStringTest1.png)  

字符串的下标 **<font color=orange>从左开始，从零开始</font>**。  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/javaString2.png)  

# 1 java String类的常用方法

## 1.1 子串 substring

```java
/**
 *
 * @param beginIndex - the beginning index, inclusive
 * @param endIndex - the ending index, exclusive.
 */
public String substring(int beginIndex,int endIndex)
```
java 获取子串方法是 **含头不含尾** 的：  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/javaStringExample1.png)  

substring的重载方法：  
```java
public String substring(int beginIndex)
//等价于
public String substring(int beginIndex, str.length())
```

## 1.2 索引 index

正向取子串的索引：  
```java
public int indexOf(String subStr, int fromIndex)
//
//
public int indexOf(String subStr) //等价于： public int indexOf(String subStr, 0)
```
在左边设置一个 查找起始索引。  
**返回值 >= 起始索引fromIndex** 。  

反向取子串的索引：  
```java
public int lastIndexOf(String subStr, int fromIndex)
```
从右往左查找，在 “右边” 设置一个 查找起始索引，因为是 **从右往左** 查找，所以一定有：   
**返回值 <= 查找起始索引fromIndex** 。

```java
public int lastIndexOf(String subStr)
//等价于
public int lastIndexOf(String subStr, str.length() )
```

## 1.3 替换 replace

替换返回的结果是替换后的字符串。**原字符串不会被修改** 。  

```java
/**
 *Parameters:
    target - The sequence of char values to be replaced
    replacement - The replacement sequence of char values
 *Returns: The resulting string
 *Throws: NullPointerException - if target or replacement is null.
 */
public String replace(CharSequence target, CharSequence replacement)
```
使用[正则表达式](https://mitrecx.github.io/2018/10/30/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F-1/)的 replaceAll 方法：  
```java
/**
* @param   regex
*          the regular expression to which this string is to be matched
* @param   replacement
*          the string to be substituted for each match
* @throws  PatternSyntaxException
*          if the regular expression's syntax is invalid
*
* @see java.util.regex.Pattern
*/
public String replaceAll(String regex, String replacement)
```

## 1.4 切割 split

```java
public String[] split(String regex, int limit)
```
按照正则表达式 regex 来切割。limit 限制 **切割的次数** 。  
1. limit 为正， 限制切割次数为（limit-1）次。  
2. limit 为负， 不限制切割次数。  
3. limit 为零， 不限制切割次数，但忽略切割后尾部的所有空字符串。  

比如：  
字符串 "boo:and:foo"   

|Regex	|Limit|	Result|note|
|--|--|--|--|
|:|	2|	{ "boo", "and:foo" }|limit不够|
|:|	5|	{ "boo", "and", "foo" }|limit足够大|
|:|	-2|	{ "boo", "and", "foo" }|不限制|
|o|	5|	{ "b", "", ":and:f", "", "" }||
|o|	-2|	{ "b", "", ":and:f", "", "" }|不限制<br/>(保留尾部空字符串)|
|o|	0|	{ "b", "", ":and:f" }|不限制<br/>不保留尾部空字符串|

```java
public String[] split(String regex)
//等价于
public String[] split(String regex, 0)
```

## 1.5 其他
1、contains 方法：  
```java
public boolean contains(CharSequence s)
```
字符串 包含 子串s 返回true， 否则 false。  

2、matches 方法：  
```java
boolean	matches(String regex)
```

3、大小写：  
```java
String	toLowerCase();

String	toUpperCase();
```

4、trim ：
```java
String	trim()
```
# 参考  
1、 [java Class String](https://docs.oracle.com/javase/7/docs/api/)  
