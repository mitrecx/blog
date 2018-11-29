---
title: Oracle 存储过程
date: 2018-09-25 09:26:26
tags: ["Oracle", "数据库"]
categories: 数据库
---

# 1 Oracle存储过程语法
```sql
CREATE [OR REPLACE] PROCEDURE 存储过程名[(参数[IN|OUT|IN OUT] 数据类型...)]
{AS|IS}
[变量声明块]
BEGIN
可执行部分;
[EXCEPTION错误处理部分]
END [过程名];
```
其中：  
可选 **关键字OR REPLACE** 表示如果存储过程已经存在，则用新的存储过程覆盖，通常用于存储过程的重建。   

**参数** 部分用于定义多个参数(如果没有参数，就可以省略)。  
参数有三种形式：IN、OUT 和 IN OUT。  
IN 按值传递，它 **不允许** 在存储过程中被重新赋值，**<font color=red>IN 参数被赋值会报错！</font>** 如果没有指明参数的形式，则默认为IN。  
**参数的数据类型** 只指明类型名即可，不需要指定宽度。**<font color=red>指定参数宽度会报错！</font>** 参数的宽度由外部调用者决定。  

**关键字 AS** 也可以写成 IS，**在此定义 局部变量**。  
可以理解为pl/sql的 declare关键字，用于声明变量。  

注：部分例子没有实际用处，只是为了说明存储过程的使用。  

----
## 1.1 利用存储过程循环插入
示例：插入100条数据到test_cus表中  
```SQL
create or replace procedure loop_insert
as
  cus_code varchar2(10);
begin
  for i in 1..100 loop
    cus_code := '1'||lpad(i,3,0);
    insert into test_cus(cus_id,cus_name,cus_age)
    values(cus_code,'存储过程',100);
  end loop;
  commit;
end;

--执行
begin
  loop_insert();
end;
```
结果：  
```
1001	存储过程	100
1002	存储过程	100
1003	存储过程	100
1004	存储过程	100
1005	存储过程	100
...
```

注：  
" := " 表示 赋值语句   
" = " 判断是否相等  
" :variableName " 表示 变量绑定  

[关于 for 循环](#id2_2)  


## 1.2 存储过程参数

以下存储过程可以正常执行：  
```sql
create or replace procedure sp_test5
(
  --不能指定参数的宽度
  para1 in out varchar2
)
as
begin
para1 :='1234567890';
end;

--调用
declare p1 varchar2(10):='a';
begin
  sp_test5(p1);
end;
```
如果把 p1 定义改为 varchar(1) :  
```sql
declare p1 varchar2(10):='a';
```
则会报错：**<font color=red>字符串缓冲区太小</font>**    
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-28_145712.png)  

以下存储过程 **能正常运行**。但如果去掉注释，则会报错，原因是 **<font color=red>不能给 IN 参数赋值</font>**。  
```sql
create or replace procedure sp_test4
(
  para1 varchar2,
  para2 out varchar2,
  para3 in out varchar2
)
as
  v_name varchar2(20);
begin
  --para1 := 'aaa';
  --para2 := 'bbb';
  v_name :=' Oracle';
  para3 := '你好';
  dbms_output.put_line('输出:'||para3||v_name);
end sp_test4;

--调用
declare p1 varchar2(20) := 'hello1';
p2 varchar2(20) := 'hello2';
p3 varchar2(20) := 'hello3';
begin
  sp_test4(p1,p2,p3);
end;
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-28_151527.png)  

再看一个例子：  
```sql
create table test_user
(
  user_id int,
  user_Name varchar2(100),
  user_Passwd varchar2(100),
  user_Age  int
)

insert into test_user values(1,'jack','xxx',23);
insert into test_user values(2,'rose','xxx',21);
insert into test_user values(3,'kiki','xxx',33);
insert into test_user values(4,'cherry','xxx',24);
commit;

--存储过程：用于更改用户年龄
create or replace procedure sp_update_age
(
  uname in varchar2,
  age in int
)
as
begin
  update test_user
     set user_age=age
   where user_name=uname;
   commit;
end sp_update_age;

--调用
begin
  sp_update_age('jack',100);
end;
```
结果：  
```
1	jack	xxx	100
2	rose	xxx	21
3	kiki	xxx	33
4	cherry	xxx	24
```

# 2 控制语句

## 2.1 if else
利用 if else 实现求一个数的绝对值：  
```sql
create or replace procedure sp_test2(x in out number)
is
begin
  if x<0 then
    begin
      x:= 0-x;
    end;
  elsif x>0 then  --注意 elsif
    begin
      x:= x;
    end;
  else
    begin
      x:= 0;
    end;
  end if;
end sp_test2;

--调用 sp_test2
declare num number;
begin
    num:= -106;
    sp_test2(num);
    dbms_output.put_line( 'num = ' || num );
end;
```
结果：  
```
num = 106
```

<h2 id="id2_2">2.2 for</h2>

for 循环示例：  
```sql
declare
x number := 100;
begin
  for i in 1..5 loop
    if mod(i,2)=0 then
      dbms_output.put_line('i: '||i||' is even.');
    else
      dbms_output.put_line('i: '||i||' is odd.');
    end if;
    x := x+1;
    dbms_output.put_line('x: '||x);
    dbms_output.put_line('');
  end loop;
  commit;
end;
```
结果：  
```
i: 1 is odd.
x: 101

i: 2 is even.
x: 102

i: 3 is odd.
x: 103

i: 4 is even.
x: 104

i: 5 is odd.
x: 105
```

## 2.3 while

while 示例：  
```sql
create or replace procedure sp_test3( i in out number)
as
begin
  while i<=100 loop
    begin
      i:=i+10;
    end;
  end loop;
end sp_test3;

--调用 sp_test3
declare t3 number := 2;
begin
  sp_test3(t3);
  dbms_output.put_line(t3);
end;
```
结果：  
```
102
```
