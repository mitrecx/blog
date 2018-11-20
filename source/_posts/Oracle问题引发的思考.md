---
title: Oracle问题引发的思考
date: 2018-10-22 15:33:18
tags:
- Oracle
- 数据库
categories: 数据库
---

问题描述：  
如下图，有两张表 emp 和 emp_history，现在想把 emp 中的数据拷贝到 emp_history 中，  
同时添加一个日期字段。两张表的主键都是id，所以只拷贝id不同的。  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/2018-10-22_154613.png)  ![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-09/2018-10-22_154736.png)  

只拷贝，不考虑主键的话：  
```sql
insert into emp_history
select e.*,to_char(sysdate,'yyyy-mm-dd') from emp e;
```

**如何判断 emp 中的数据在 emp_history 中是否存在？**(id相同的数据)   
我首先想到的是 when then , 然后想用一个存储过程实现，如果存在就不插入，不存在就插入。  
但是当我无法实现的时候，我才想到了 **exists** 。   
事实上，用 **exists** 可以很容易实现查找出重复 id 的数据：  
```sql
select e.*,to_char(sysdate,'yyyy-mm-dd')
from emp e
where not exists ( select 1 from emp_history eh
                    where eh.id=e.id);
```

问题解决：  
```sql
insert into emp_history
select e.*,to_char(sysdate,'yyyy-mm-dd')
from emp e
where not exists ( select 1 from emp_history eh
                     where eh.id=e.id );
```
