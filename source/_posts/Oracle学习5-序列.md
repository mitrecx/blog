---
title: Oracle-学习5-序列
date: 2018-11-26 19:54:32
tags:
- 数据库
- Oracle
categories: 数据库
---

# 创建序列 sequence  

```sql
CREATE SEQUENCE 序列名

　　[INCREMENT BY n]

　　[START WITH n]

　　[{MAXVALUE/ MINVALUE n| NOMAXVALUE}]

　　[{CYCLE|NOCYCLE}]

　　[{CACHE n| NOCACHE}];
```
1、   
**<font color='#00b33c'>Increment by n</font>** ， 步长为n  
**默认为 1**

2、   
**<font color='#00b33c'>Stat with n</font>** ， 开始值为n  
**默认为 1**  

3、  
**<font color='#00b33c'>Maxvalue n</font>** ， 最大值  
**<font color='#00b33c'>Minvalue n</font>** ，  最小值  
**<font color='#00b33c'>NOMAXVALUE</font>** ， 无最值(最大：10的27次方，最小：-10的26次方)    
**默认 无最值**  

4、   
**<font color='#00b33c'>CYCLE</font>** 循环，递增或递减过最值时，回到起始值循环  
**<font color='#00b33c'>NOCYCLE</font>** 非循环，递增或递减过最值时，报错。   
**默认 非循环**  

5、   
**<font color='#00b33c'>CACHE n</font>** 缓冲， 定义存放序列的内存块的大小。  
**默认为 20**   

# 访问序列  

访问序列的下一个值：  
```sql
select 序列名.nextVal from dual;
```

访问序列当前值：  
```sql
select 序列名.currVal from dual;
```

注：  
新创建的 序列， 只能 在nextVal一次之后，才能使用currVal。  

# 修改序列  
**Oracle 无法直接修改序列的 起始值**    
**改变序列的初始值** 只能通过 **先删除序列** 再重建序列来实现    

修改序列的步长为 1000 ：  
```sql
alter sequence 序列名
increment by 1000;
```

最值，循环，缓冲 都能被修改。  
值得注意的是，**循环** 和 **缓冲** 必须满足下面不等式：  
```
cache < CEIL(maxValue-minValue)/ABS(increment)
```

如果想要把序列的当前值 +1000，可以先修改步长为1000，再nextVal，再把步长改为原值。   

# 删除序列  

```sql
drop sequence 序列名;
```
删除之后，序列将不能被引用。  
