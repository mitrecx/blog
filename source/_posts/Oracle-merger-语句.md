---
title: Oracle merger 语句
date: 2018-12-11 10:25:50
tags: Oracle
categories: Oracle
---

Oracle 对 **merge** 语句的介绍：  
Use the **<font color=red>MERGE</font>** statement to select rows from one or more sources for update or insertion into a table or view.  
You can specify conditions to determine whether to update or insert into the target table or view.  
This statement is a convenient way to combine multiple operations.  

如果数据库中存在数据就 update，如果不存在就insert 。
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

**把 test_cus1 表中的数据 插入 cus_his 表中**  
如果 cus_his 已经有此用户记录，则更新 cus_his 表中此用户信息；  
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
