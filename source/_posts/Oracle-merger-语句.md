---
title: Oracle merger 语句
date: 2018-12-11 10:25:50
tags: Oracle
categories: 数据库
---

Oracle 对 **merge** 语句的介绍：  
Use the **<font color=red>MERGE</font>** statement to **select rows from one or more sources for update or insertion** into a table or view.  
You can specify conditions to determine whether to **update or insert into** the target table or view.  
This statement is a convenient way to combine multiple operations.  

例如，数据库有两张表 new 和 old，现在把 new 里的数据合并(插入)到 old 里。    
如果是 old 表中 **已存在的记录就 update，如果不存在就 insert** 。  

当我们有 insertOrUpdate 这种需求的时候，使用 merge 会很容易。  

看一个例子：  
```sql
--客户表
select * from test_cus1;
--客户历史表
select * from cus_his ;
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2018-12-11_101429.png)  

**把 test_cus1 表中的数据 插入 cus_his 表中:**  
如果 cus_his 已经有此用户记录(根据 cus_id 确定唯一客户)，则更新 cus_his 表中此用户信息；  
如果 cus_his 没有此用户，则插入 cus_his 表中。  

```sql
merge into cus_his his
  using test_cus1 t
  on( his.cus_id = t.cus_id)
  when matched then
    update set
       his.cus_age=t.cus_age,
       his.cus_name=t.cus_name
  when not matched then
    insert (his.cus_id,his.cus_name,his.cus_age)
    values(t.cus_id,t.cus_name,t.cus_age);
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2018-12-11_102252.png)  
结果分析：  
cus_his 中已经存在的 “蚂蚁大哥”(cus_id=1001) 被 更新，  
cus_his 中不存在的(cus_id=1002,1003,1004) 3 条用户记录 被插入 cus_his。
