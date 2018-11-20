---
title: oracle存储过程
date: 2018-09-25 09:26:26
tags:
- Oracle
- 数据库
categories: 数据库
---
```sql
CREATE [OR REPLACE] PROCEDURE 存储过程名[(参数[IN|OUT|IN OUT] 数据类型...)]
{AS|IS}
[说明部分]
BEGIN
可执行部分;
[EXCEPTION错误处理部分]
END [过程名];
```
其中：  
可选 **关键字OR REPLACE** 表示如果存储过程已经存在，则用新的存储过程覆盖，通常用于存储过程的重建。   

**参数** 部分用于定义多个参数(如果没有参数，就可以省略)。  
参数有三种形式：IN、OUT和IN OUT。如果没有指明参数的形式，则默认为IN。  

**关键字AS** 也可以写成IS，后跟过程的说明部分，可以 **在此定义过程的局部变量**。  

示例：插入100条数据到test_cus表中  
```SQL
create or replace procedure loop_insert
as
  cus_code varchar2(10);
begin
for i in 5..100 loop
  cus_code := '1'||lpad(i,3,0);
  insert into test_cus(cus_id,cus_name,cus_age) values(cus_code,'存储过程',100);
 end loop;
 commit;
end;
```

执行：  
```sql
begin
  loop_insert();
end;
```
