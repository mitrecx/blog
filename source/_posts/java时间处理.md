---
title: java时间处理
date: 2018-11-01 09:04:29
tags: java
categories: java
---
# 0. 前言
过去世界各地原本各自订定当地时间，但随着交通和电讯的发达，各地交流日益频繁，不同的地方时间，造成许多困扰。  
于是，在1884年的国际会议上制定了全球性的标准时，明定以英国伦敦 **格林威治** 这个地方为 **零度经线** 的起点，并以地球由西向东每24小时自转一周360°，订定每隔经度15°，时差1小时。  

三个时间标准：  
1、 **格林威治标准时间** GMT (Greenwich Mean Time):  
全球都以格林威治的时间作为标准来设定时间。   
2、 世界协调时间UTC (Coordinated Universal Time):  
UTC的本质强调的是比GMT更为精确的世界时间标准。  
3、 CST时间:  
CST却同时可以代表如下 4 个不同的时区:  
1. Central Standard Time (USA) UT-6:00  
2. Central Standard Time (Australia) UT+9:30  
3. **China Standard Time UT+8:00** 北京时间  
4. Cuba Standard Time UT-4:00 古巴

注：  
时间比较一般说更早，更晚。  
以1970 年 1 月 1 日 00:00:00 GMT 到目前的毫秒数作为参考的话，  
可以说 **早** 一点的时间 **小**，**晚** 一点的时间 **大**。  
比如，2018>1995  

# 1. java.util.Date 类

说到 java 时间，大家都知道 **Date** 类型。  
Date 有许多方法都已经不建议使用了(**deprecated**)，但是 Date 依然是一个最基本的时间类。  

## 1.1 Date 常用的方法
Date 常用的方法有以下三个：  

1、获取当前系统时间
```java
Date date=new Date();
```
<code>System.out.println(date);</code> 结果：  
```
Thu Nov 01 10:58:58 CST 2018
```
2、 获取时间到1970 年 1 月 1 日 00:00:00 GMT 的毫秒值
```java
//long getTime()
long timeMs=date.getTime();
```
获取当前系统时间毫秒值：  
```java
long begin=System.currentTimeMillis();
```


3、 两个时间比较大小  
```java
//int compareTo(Date anotherDate)
date1.compareTo(date2)
```
结果：  
**date1** 比 date2 **小**(早)，则返回 **-1**    
date1 比 date2 大(晚)，则返回 1   
相等，返回0    

## 1.2 Date 和 String 相互转换  
**java.text.SimpleDateFormat** 格式化时间      
```java
//SimpleDateFormat(String pattern)
SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
```
SimpleDateFormat 时间格式(pattern)的含义 :  
yyyy 四位年。  
MM 月。  
dd 天。  
**HH** 24小时制时，hh是12小时制时。  
mm 分。  
ss 秒。  
~~SSS 毫秒~~    

### 1.2.1 format 将Date转为String
```java
Date date = new Date();
System.out.println("date: "+date);
//时间格式
SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String sdate=format.format(date);
System.out.println(sdate);
```
结果：  
```
date: Thu Nov 01 14:52:47 CST 2018
2018-11-01 14:52:47
```

### 1.2.2 parse 将String转为Date
在上一小节中，format 把 Date 转为 String 没有问题。  

**<font color=red>parse 把 String 转为 Date 时</font>**，如果 String 不符合 **SimpleDateFormat** 的格式，将会抛出 **<font color=red>ParseException</font>** 异常。所以字符串转Date需要try-catch。    

```java
String s="2018-11-11 9:9:9:123";
try {
  Date date2=format.parse(s);
  System.out.println("date2: "+date2);
} catch (ParseException e) {
  e.printStackTrace();
}
```
结果：  
```
date2: Sun Nov 11 09:09:09 CST 2018
```

# 2. 时间的计算  
抽象类(abstract class) **<font color='#9900cc'>Calendar</font>** 提供了丰富的计算时间的方法。  

## 2.1 初识Calendar  
一个实例：  
```java
SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

//GregorianCalendar 继承自 Calendar，实现了所有抽象方法
Calendar calendar=new GregorianCalendar();
//Calendar calendar=Calendar.getInstance();

//获取Date
Date date=calendar.getTime();
System.out.println(format.format(date));
```
结果：  
```
2018-11-01 15:23:48
```

## 2.2 Calendar 计算时间  
### 2.2.1 天数加减计算：  
```java
public static void main(String[] args) {
  SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  Calendar calendar = new GregorianCalendar();

  // 34天前
  calendar.add(Calendar.DAY_OF_YEAR, -34);
  System.out.println(format.format(calendar.getTime()));
  // 34天后
  calendar.add(Calendar.DAY_OF_MONTH, 34);
  System.out.println(format.format(calendar.getTime()));
  // 64天后
  calendar.add(Calendar.DAY_OF_WEEK, 64);
  System.out.println(format.format(calendar.getTime()));
}
```
结果如下，当前时间是 2018-11-01 16:00:42 :  
```
2018-09-28 16:00:42
2018-11-01 16:00:42
2019-01-04 16:00:42
```
分析：  
```java
void add(int field, int amount)
```
**<font color='#e68a00'>field</font>** 是日期字段类型， **<font color='#e68a00'>amount</font>** 表示时间的偏移总量。  
DAY_OF_YEAR，DAY_OF_MONTH，DAY_OF_WEEK在上面的例子中效果是相同的。  
原因在于 **add** 的内部实现最终处理是一样的。  

**<font color='#e68a00'>field</font>** 可用的名称如下，进行时间的运算离不开这些 field ：    
```java
private static final String[] FIELD_NAME = {
    "ERA",
    "YEAR",
    "MONTH",
    "WEEK_OF_YEAR",
    "WEEK_OF_MONTH",
    "DAY_OF_MONTH",
    "DAY_OF_YEAR",
    "DAY_OF_WEEK",
    "DAY_OF_WEEK_IN_MONTH",
    "AM_PM",
    "HOUR",
    "HOUR_OF_DAY",
    "MINUTE",
    "SECOND",
    "MILLISECOND",
    "ZONE_OFFSET",
    "DST_OFFSET"
};
```

### 2.2.2 年月加减计算  
还是 add 函数，只是 field_name 改了。  
```java
public static void main(String[] args) {
  SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  Calendar cal=new GregorianCalendar();
  System.out.println("当前时间："+format.format(cal.getTime()));

  cal.add(Calendar.MONTH, 3);
  System.out.println("3个月后："+format.format(cal.getTime()));
  cal.add(Calendar.MONTH, -12);
  System.out.println("12个月前："+format.format(cal.getTime()));
  cal.add(Calendar.YEAR, -10);
  System.out.println("10年前："+format.format(cal.getTime()));
}
```
结果：  
```
当前时间：2018-11-01 16:44:36
3个月后：2019-02-01 16:44:36
12个月前：2018-02-01 16:44:36
10年前：2008-02-01 16:44:36
```

### 2.2.3 其他操作

1、 Calendar 的 **get** 和 **set** 方法
```java
//获取日期的每部分
int year =calendar.get(Calendar.YEAR);   
int month=calendar.get(Calendar.MONTH)+1;  //月份从0开始
int day =calendar.get(Calendar.DAY_OF_MONTH);   
int hour =calendar.get(Calendar.HOUR_OF_DAY);   
int minute =calendar.get(Calendar.MINUTE);   
int second =calendar.get(Calendar.SECOND);  
//日期在年的第几周
int weekOfYear = calendar.get(Calendar.WEEK_OF_YEAR);
//周几，1-周日，2-周一，3-周二... 7-周六
int dayOfWeek=calendar.get(Calendar.DAY_OF_WEEK);
```
上面列举了get方法。 **set** 用于设置时间，不再举例。   

2、获取月的总天数
```java
int day=calendar.getActualMaximum(Calendar.DAY_OF_MONTH);
```

3、**两个日期之间相隔的天数**  
```java
// date1 和 date2 为 Date类型
long day=(date1.getTime()-date2.getTime())/(24*60*60*1000);
```
4、9月份第一个星期一是几号  
```java
public static void main(String[] args) {
  SimpleDateFormat format=new SimpleDateFormat("yyyy-MM-dd");
  Calendar calendar=new GregorianCalendar();

  //设置时间为9月1号
  calendar.set(Calendar.MONTH, Calendar.SEPTEMBER); //8
  int day=1;
  calendar.set(Calendar.DAY_OF_MONTH, day);
  System.out.println(format.format(calendar.getTime()));
  //遍历直到第一个星期一
  while(calendar.get(Calendar.DAY_OF_WEEK)!=Calendar.MONDAY) {
    calendar.set(Calendar.DAY_OF_MONTH, ++day);
  }
  System.out.println(format.format(calendar.getTime()));
  System.out.println(calendar.getTime());
}
```
结果:  
```
2018-09-01
2018-09-03
Mon Sep 03 18:52:07 CST 2018
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-01_184844.png)  
