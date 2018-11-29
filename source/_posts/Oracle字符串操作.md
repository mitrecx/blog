---
title: Oracle字符串操作和时间操作
date: 2018-11-21 10:18:49
tags: ["数据库","Oracle"]
categories: 数据库
---

# 1 Oracle 字符串类型

字符串类型的数据可依 **编码方式** 分成 **数据库字符集**（CHAR/VARCHAR2/CLOB/LONG)和 **国际字符集**(NCHAR/NVARCHAR2/NCLOB)两种。  
数据库中的字符串数据都通过字符集将字符转换为二进制，才存储到数据块中。  
通过不同的编码集转换，即便是相同的字符，也可能会转换成不同的二进制编码。这也是产生乱码的原因。  
数据库的编码格式一般是在创建数据库时指定的。也可以在数据库建立之后修改其编码。  
```sql
--字符集查询
select userenv('language') from dual;
```

字符串类型 依据 **存储方式** 分为 **固定长度类型**（CHAR/NCHAR) 和 **可变长度类型**（VARCHAR2/NVARCHAR2)两种.

1. 固定长度：是指虽然输入的字段值小于该字段的限制长度，但是实际存储数据时，会先自动向右补足空格后，才将字段值的内容存储到数据块中。(浪费空间，但是存储效率较可变长度类型要好)  
例：char的长度是固定的，比如，定义了char(20),即使插入abc，不足二十个字节，数据库也会在abc后面自动加上17个空格，以补足二十个字节。  

2. 可变长度：当输入的字段值小于该字段的限制长度时，直接将字段值的内容存储到数据块中，而不会补上空白。(节省数据块空间)  


Oracle 字符串类型：    
1. char(size) / nchar(size) ：定长，字节/字符。最长2000字节。  
2. varchar(size) / **varchar2(size)** ：不定长（变长），字节。最长4000字节。  
3. nvarchar(size) / nvarchar2(size) ：不定长，字符  
4. long 和 clob ：不定长。long 2GB，clob 4 GB。  

# 2 Oracle 字符串处理函数

1. concat 和 ||，拼接字符串
2. length
3. upper，lower，和 initcap(各个单词首字母大写)

4. substr
5. trim，ltrim，rtrim
6. instr  
7. lpad，rpad

介绍一下 4,5,6,7 ：  

**substr** 选取子串的起始位置是 1，不是 0。（但是，写0的效果和1相同，都会从起始位置选取）
```sql
--substr('xxx',start,size)

select substr('hello world',1,4) from dual ;
--结果： 'hell'
```


**trim** 只对 **两端** 有效，去除两端空白，或者去除两端某一个指定字符。
```sql
--去除 两端 空白
select trim('   mitre  cx   ') from dual;

--去除 左端 空白
select ltrim('   mitre  cx   ') from dual;

--去除 右端 空白
select rtrim('   mitre  cx   ') from dual;

--去除 两端  字符e
select trim('e' from 'eeemitre  eeecxee') from dual;
```

**instr** (index string的缩写) 字符查找函数，查找到指定字符串，返回指定子串的下标(注意从1开始)，未找到返回0。  

格式一：instr(源字符串, 目标字符串)    
格式二： instr(源字符串, 目标字符串[, 起始位置[, 匹配序号]])  


```sql
select instr('helloworld','wo') from dual;
--返回结果：'6'    
--即“w”开始出现的位置

select instr('helloworld','o',1,2) from dual;
--结果：'7'
--从起始位置 1 开始查找，找第 2 个 ‘o’ 出现的位置
```

lpad 在 字符串的左边补位。  
rpad 在 字符串的右边补位。  
```sql
select  lpad('world',13,'000 ') from dual;  
--结果：'000 000 world'
--在 'world' 的左边补位 '000 ', 补满 13位

select  lpad('world',3,'000 ') from dual;
--结果：'wor'
--总位数少于原始字符串位数，则保留左侧(rpad也是保留左侧)
```

# 3 时间类型 date

获取当前系统时间：  
```sql
select sysdate from dual;
--结果：'2018/11/22 9:28:20'
```
## 3.1 时间和字符串

**时间转字符串 to_char(timestamp, timeFormat)**：   
YYYY	年  
MM	月份 (01-12)  
DD	一个月里的日子(01-31)  
D	一周里的日子(1-7；SUN=1)
day 星期几  

HH	一天的小时数 (01-12)  
HH12	一天的小时数 (01-12)  
HH24	一天的小时数 (00-23)  
MI	分钟 (00-59)  
SS	秒 (00-59)  
[更多格式.](https://www.cnblogs.com/aipan/p/7941917.html)  

例：  
```sql
select to_char(sysdate,'yyyy-mm-dd hh24:mi:ss , day') from dual;
--结果：'2018-11-22 10:13:04 , 星期四'

-- 本月的天数
SELECT to_char(last_day(SYSDATE),'dd') days FROM dual;
--结果：'30'
--注：last_day(time):返回指定日期所在月份的最后一天
```

**字符串转时间 to_date(charType, timeFormat)**:  
```sql
select to_date('2018-11-22','yyyy-mm-dd') from dual;
--结果：'2018/11/22'
```

## 3.2 时间的操作

在介绍时间操作前，先了解 3 个函数：  
1. round(num [,decimal_scale]) 函数: 四舍五入。     
2. floor(num) 函数 : 地板，取小于等于数值 num 的最大整数。  
3. ceil(num) 函数 ：天花板，取大于等于数值 num 的最小整数。  

示例：  
```sql
select round(123.456,2) from dual; -- '123.46'
select round(123.456) from dual; -- '123'

select floor(123.456) from dual; -- '123'
select ceil(123.456) from dual; -- '124'

select floor(-1.2) from dual; -- '-2'
select ceil(-1.2) from dual; -- '-1'
```

**计算时间差：时间相减**   
```sql
--两个日期间的天数
select sysdate - to_date('2018-10-23','yyyy-mm-dd') from dual; -- '30.4204282407407'

select floor(sysdate - to_date('2018-10-23','yyyy-mm-dd')) from dual; -- '30'
```

```sql
--求某天是星期几
select to_char(to_date('2018-11-22','yyyy-mm-dd'), 'day') from dual;  -- '星期四'
```

**trunc 函数 --截取函数**   
truncate, 截短，删节。

trunc函数有两种形式：  

1. trunc(number [,decimal_scale]) ，
截取数字 , decimal_scale 保留小数点后面的位数。忽略它则截去所有的小数部分。  
若 decimal_scale 为负，则向整数位截取补零。

2. trunc(datestamp [,time_scale]) ，
截取时间 , timeFormat 截取时间保留的位数。忽略默认截取到日。  
截取的结果是日期：年/月/日。

trunc 函数 示例：  
```sql
select trunc(123.456,2)from dual; -- '123.45'
select trunc(123.456,1)from dual; -- '123.4'
select trunc(123.456)from dual; -- '123'

--decimal_scale为负，向整数位截取补零
select trunc(123.456,-1)from dual; -- '120'
select trunc(123.456,-2)from dual; -- '100'

--截取时间
--默认截取，等价于 trunc(sysdate, 'dd')
select trunc(sysdate) from dual; --结果：'2018/11/22'
--返回当年第一天
select trunc(sysdate,'YYYY')from dual; --结果：'2018/1/1'
--返回当月第一天
select trunc(sysdate,'MM')from dual; --结果：'2018/11/1'
--返回当前星期的第一天(星期天)
select trunc(sysdate,'D')from dual; --结果：'2018/11/18'
```

Oracle其他 有关时间的方法：  
```sql
--在系统时间基础上延迟5月
select add_months(sysdate,-5) from dual t; --结果：'2018/6/22 11:09:36'

--下个星期一的日期
select next_day(SYSDATE,2) from dual --结果：'2018/11/26 11:11:50'
-- 1  2  3  4  5  6  7
--日 一 二 三 四 五 六
```  
