---
title: java正则表达式-预查
date: 2018-11-02 14:45:44
tags: java
categories: java
---
应用场景：  
解析报文(xml文件)，去除字段的空白字符(空白字段，字段首尾空白)。

思想：   
利用正则表达式的 **预查** 定位到各个字段，并匹配到字段的空白字符，替换之。  

实现如下：   
```java
public static void main(String[] args) {
  StringBuilder builder=new StringBuilder();
  builder.append("<name type='string'>cx  </name>\n");
  builder.append("<age>     </age>\n");
  builder.append("<hight> 1.72 </hight>\n");
  builder.append("<weight>   56</weight>\n");
  builder.append("<sex> m a n </sex>");
  System.out.println(builder.toString());
  System.out.println("=====去除空串字段的空白字符======");
  String s=builder.toString().replaceAll("(?<=>)\\s+(?=<)", "");
  System.out.println(s);
  System.out.println("=====去除所有字段的 首尾空白字符======");
  //除去空白字段，和首标签后的空白字符
  String s1=builder.toString().replaceAll("(?<=>)\\s+(?=(\\w|<))", "");
  System.out.println(s1);
  //除去尾标签前的空白字符
  s1=s1.replaceAll("(?<=\\w)\\s+(?=<)", "");
  System.out.println(s1);
}
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-02_144254.png)  

分析：  
正向预查 **strRE(<span style="background:#ffb3d9;">?=</span>regex)**，查出前面的strRE，regex只做参考不做匹配。   
反向预查 **(<span style="background:#ffb3d9;">?<=</span>regex)strRE**，查出后面的strRE，regex只做参考不做匹配。  

# 1. 正向预查
利用正则表达式正向预查匹配出 **后面带数字Windows**，如下：    
**<font color=red>Windows</font>** 1.03 and **<font color=red>Windows</font>** 2.0 fisrt Released in 1985 and 1987 respectively.  
**<font color=red>Windows</font>** 95 and **<font color=red>Windows</font>** 98 are the successor.  
Then **<font color=red>Windows</font>** 2000 and **Windows** Xp appeared.  
**Windows** Vista is the Latest version of the family.  

正则表达式：  
**Windows (<span style="background:#ffb3d9;">?=</span>[\d.]+\b)**    
\d 表示匹配一个数字  

结果：  
后面带有数字的 Windows 被匹配出来(只匹配Windows，不含数字)，不带数字Windows的不匹配。   

可以这样理解匹配过程：  
1. 先进行普通匹配：Windows ([\d.]+\b)
2. 然后从匹配结果中将 **子模式** 内的文本<code>([\d.]+\b)</code>排除掉  

# 2. 反向预查
匹配出 “CNY:” 后面的金额：   
CNY: **<font color=red>128.04</font>**    
USD: 22.5  
USD: 23.5  
HKD: 1533.5  
CNY: **<font color=red>23.78</font>**    

正则表达式：  
**(<span style="background:#ffb3d9;">?<= </span>CNY: )\d+\.\d+**   


可以这样理解上面的匹配过程：
1. 先进行普通匹配：(CNY: )\d+\.\d+
2. 然后从匹配文本中将 **子模式** 内的文本<code>(CNY: )</code>排除掉。
