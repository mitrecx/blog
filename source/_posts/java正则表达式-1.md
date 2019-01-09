---
title: java正则表达式-1
date: 2018-10-30 09:21:54
tags: 正则表达式
categories: java
---
**java.util.regex** 下有两个类，**<font color='#00e600'>Pattern</font>** 和 **<font color='#3399ff'>Matcher</font>** 。    
# 1. 抛砖引玉  **<font color='#00e600'>Pattern.matches</font>**

**Pattern.matches(regex,str)** 要求 **字符串str** <font color='#cc00cc'>完全符合</font> **匹配规则(字符串)regex** ，返回结果才为 true。    
```java
public static void main(String[] args) {
		String str="A compiled representation of a regular expression.";
		String regex=".*re.*";
		boolean b=Pattern.matches(regex, str);
		System.out.println("是否包含字符串re: "+b);

		//Pattern.matches(regex, str) 等价于Pattern.compile(regex).matcher(str).matches()
		boolean b2=Pattern.compile(regex).matcher(str).matches();
		System.out.println(b2);

		System.out.println(Pattern.matches("re", "A compiled representation of a regular expression."));//false
		System.out.println(Pattern.matches("\\d+","123")); //true
		System.out.println(Pattern.matches("\\d+","123a")); //false
		System.out.println(Pattern.matches("\\d+","123a456")); //false
	}
```
正则表达式，就是一个指定匹配规则的字符串，这个字符串(正则表达式)必须被编译成 **<font color='#e60000'>Pattern类的实例</font>** 才能创建 **<font color='#3399ff'>Matcher 对象</font>**。正则表达式先编译成 Pattern实例 再创建 Matcher对象有一个好处：多个 matcher 可以共享一个 pattern，达到复用的效果。    
如下，引用 java API 提供的[例子](https://docs.oracle.com/javase/7/docs/api/):    
```java
Pattern p = Pattern.compile("a*b");
Matcher m = p.matcher("aaaaab");
boolean b2 = m.matches(); //true
```
下面的写法与上面等价，其内部实现就是上面三行代码，只是 **一个匹配的简写** 罢了：    
```java
boolean b1 = Pattern.matches("a*b", "aaaaab");  //true
```

# 2. Groups and capturing 捕获组
通过 **从左到右** 数开头圆括号，对捕获组进行编号。例如下面的正则表达式 **捕获组** ：  
**( (A) (B (C)) )**  
1 &emsp;((A)(B(C)))  
2 &emsp;(A)  
3 &emsp;(B(C))  
4 &emsp;(C)    
0 &emsp;整个字符串，((A)(B(C)))   

捕获组的例子：  
```java
public static void main(String[] args) {
  String str="for example, there are four such groups";
  String regex="((.*)(,)(.*))";
  Pattern pattern=Pattern.compile(regex);
  Matcher m=pattern.matcher(str);
  if(m.find()) {
    System.out.println("start: "+m.start());
    System.out.println("end: "+m.end());
    System.out.println("group  : "+m.group());
    System.out.println("group等价于: "+"str.substring(m.start(),m.end())");
    System.out.println(str.substring(m.start(),m.end()));
    System.out.println("------");
    System.out.println("group1 : "+m.group(1));
    System.out.println("group2 : "+m.group(2));
    System.out.println("group3 : "+m.group(3));
    System.out.println("group4 : "+m.group(4));
    System.out.println("group0 : "+m.group(0));
  }else {
    System.out.println("字符串与正则表达式 不完全匹配");
  }
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/2018-10-30_143437.png)   

Matcher 的 **<font color='#e65c00'>find() 函数</font>** 的两个特点：   
1、 如果正则表达式里 **包含** 分组。那么，**find()函数** 会对 **str** 与 **regex** 做完全匹配。若匹配上，返回 **true**，此时可以通过 **group(int i)** 来访问每一个组；若未匹配上，返回 **false** 。  
2、 如果正则表达式 **不包含** 分组。那么，**find()函数** 会对 **str** 与 **regex** 做 **不完全** 匹配。即只要 str 中包含 regex 就会返回 true。此时可以通过 **group()** 来访问匹配到的 regex (注：str中可能有多个regex，第一次find只能获取到第一个匹配，第二次find获取到第二个，以此类推)，regex在str中的起始位置可以通过 **start() 和 end()** 获得。   

**<font color='#e65c00'>find() 函数</font>** 的第 2 个特点，看例子会更清楚：  
```java
public static void main(String[] args) {
  String str1="A compiled representation of a regular expression.";
  String regex1="re";
  Pattern p=Pattern.compile(regex1);
  Matcher m1=p.matcher(str1);
  if(m1.find()) {
    System.out.println("m1.start: "+m1.start());
    System.out.println("m1.end: "+m1.end());
    System.out.println("m1.group(): "+m1.group());
    System.out.println(str1.substring(m1.start(), m1.end()));//与上等价写法

    System.out.println("m1.group(0): "+m1.group(0)); // group(1..n)会报错
    System.out.println("=====");
  }
}
```
结果：  
```
m1.start: 11
m1.end: 13
m1.group(): re
re
m1.group(0): re
=====
```
如果把if 换成 while， 结果：  
```
m1.start: 11
m1.end: 13
m1.group(): re
re
m1.group(0): re
=====
m1.start: 14
m1.end: 16
m1.group(): re
re
m1.group(0): re
=====
m1.start: 31
m1.end: 33
m1.group(): re
re
m1.group(0): re
=====
m1.start: 42
m1.end: 44
m1.group(): re
re
m1.group(0): re
=====
```

分析：  
" **A compiled <font color='#ff0000'>re</font>p<font color='#ff0000'>re</font>sentation of a <font color='#ff0000'>re</font>gular exp<font color='#ff0000'>re</font>ssion.** "    
这句话有4个 **re** ，但是第一次 find ，group() 只能匹配到下标为[11,13)的第一个字符串。后面3个 需要多次find 获得。  


看一个实际应用的例子：  
把字符串 “$year$ $month$ $day$” 中的年月日换成真实的年月日。  
```java
public static void main(String[] args) {
  Map<String,String> map=new HashMap<String,String>();
  map.put("year", "2018");
  map.put("month", "10");
  map.put("day","30");
  String str="$year$ $month$ $day$";
  //String regex="\\$[^\\$]+\\$";
  String regex="\\$.*?\\$";

  Pattern p=Pattern.compile(regex);
  while(true) {
    Matcher m=p.matcher(str);
    if(!m.find())
      break;
    String group=m.group();
    String param=group.replaceAll("\\$","");
    //System.out.println(param);
    str=str.replace("$"+param+"$",map.get(param));
  }
  System.out.println(str);
}
```
结果：  
```
2018 10 30
```
分析：  
1. **regex="\\\$.*?\\\$"** 问号表示 **非贪婪匹配** ，默认是贪婪匹配。  
2. **regex="\\\$[^\\\$]+\\\$"** 也可以完成此例，这是参照公司框架代码的写法。  
3. 此例关键在于，循环多次find，每次把第一个替换掉，这样可以遍历到所有的匹配到的字符串。  

# 3. Pattern 的方法-split  
**<font color='#00b3b3'>String[]	split(CharSequence input)</font>**  
String 实现了 CharSequence 接口，因此可以给split传一个 String 参数。  

```java
Pattern p=Pattern.compile("\\++");
String[] str=p.split("1+1+2+3");
```
结果：  
```
str[0]=1
str[1]=1
str[2]=2
str[3]=3
```
**String 也有一个 split 方法** ：   
**<font color='#0059b3'>String[] split(String regex)</font>**   
# 4. 附录  
## 4.1 常用正则表达式
|规则|	正则表达式语法     |
|---|---|---|  
|一个或多个汉字|	^[\u0391-\uFFE5]+$ |
|邮政编码|	^[1-9]\d{5}$|
|邮箱|	^[a-zA-Z_]{1,}[0-9]{0,}@(([a-zA-z0-9]-*){1,}\.){1,3}[a-zA-z\-]{1,}$|
|用户名（字母开头 + 数字/字母/下划线）|	^[A-Za-z][A-Za-z1-9_-]+$|
|手机号码	| ^1[3 &#124;4&#124;5&#124;8][0-9]\d{8}$ |
|URL|	^((http &#124;https)://)?([\w-]+\.)+[\w-]+(/[\w-./?%&=]*)?$ |
|18位身份证号|	^(\d{6})(18&#124;19&#124;20)?(\d{2})([01]\d)([0123]\d)(\d{3})(\d&#124;X&#124;x)?$|  

## 4.2 正则表达式语法

[参见](https://www.cnblogs.com/lzq198754/p/5780340.html)   
