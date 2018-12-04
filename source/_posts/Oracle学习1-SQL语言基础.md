---
title: Oracle-学习1-SQL语言基础
date: 2018-11-06 11:18:15
tags: ["Oracle","数据库"]
categories: 数据库
---
先建两张表：部门表，员工表。    
```sql
create table DEPARTMENT
(
  department_no   VARCHAR2(10) not null,
  department_name VARCHAR2(100)
)
-- 添加主键约束
alter table department
add constraint pk_dept_no primary key(department_no);

comment on table department
is '部门表';
comment on column department.department_no
is '部门编号';
comment on column department.department_name
is '部门名称';
```

```sql
create table EMP
(
  emp_id        VARCHAR2(4) not null,
  emp_name      VARCHAR2(100),
  hire_date     VARCHAR2(10),
  department_no VARCHAR2(10)
)

--修改字段 emp_id名称为 emp_no
alter table emp
rename column emp_id to emp_no;

--添加 外键约束（普通外键约束）
alter table EMP
  add constraint FK_DEPT_NO foreign key (department_no)
  references department (department_no);

--删除约束条件
alter table emp
drop constraint fk_dept_no;

--重新添加 外键约束（级联外键约束）
  alter table EMP
    add constraint FK_DEPT_NO foreign key (department_no)
    references department (department_no) on delete cascade;

comment on table emp
is '员工表';
```
两种形式的外键约束：   
1、普通外键约束（如果存在子表引用父表主键，则无法删除父表记录）  
2、级联外键约束（可删除存在引用的父表记录，而且同时把所有有引用的子表记录也删除）

创建结果：  
```sql
select tabs.table_name "表名", comments.comments, obj.created "表创建时间", obj.last_ddl_time "表修改时间"
from user_tables tabs left join user_tab_comments comments on(comments.TABLE_NAME=tabs.table_name)
left join user_objects obj on(tabs.table_name=obj.object_name)
order by obj.last_ddl_time desc;
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_151125.png)  

# 1 where 子句

在 Oracle ，任何值 **<font color=red>都不能 "=NULL"</font>** ，只能用 "is NULL" 或 "is NOT NULL"。  

空值函数 NAV 和 NVL2 ：  
nvl(str, replace_with) , 若 str 为空，则取 replace_with 的值。否则不变。   
nvl2(str, str2, replace_with),  若 str 为空，则取 replace_with 的值。否则取 str2 的值。  

为了方便测试，给 **emp表** 添加一个 salary 字段：  
```sql
alter table emp
  add salary number(10,2);
```

```sql
select * from emp;
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_154406.png)  

测试：若工资为空，则置为0，否则置为 10000。  
```sql
select nvl2(salary,10000,0)
  from emp;
```
结果：  
```
	10000
	10000
	10000
	0
```

where 子句中的 **比较运算符** ：  
<code> &gt; 大于， &gt;= 大于等于 </code>  
<code> &lt; 小于， &lt;= 小于等于 </code>  
<code> = 等于， &lt;&gt; 不等于 </code>  

查看在 2018年1月1日 之前 入职的员工：  
```sql
select * from emp
 where to_date(hire_date,'yyyy-mm-dd') < to_date('2018-1-1','yyyy-mm-dd');
```

where 子句中的 **逻辑运算符** ：   
<code> not </code>，如果原条件为真，则得到假。如果原条件为假，结果为真。  
<code> and </code>，如果左右两个条件都为真，则得到的值就为真。    
<code> or </code>，  只要左右两个条件有一个为真，则得到的值就为真。  

**注意：<font color=red> and 的优先级高于 or。</font>**  

Oracle 运算符的优先级(由高到低)：  
1. 算术运算符(即‘+’,‘-’，‘\*’,‘/’)
2. 连接运算符（即‘||’）
3. 比较运算符（即‘&gt;’，‘&gt;=’，‘&lt;’，‘&lt;=’，‘&lt;&gt;’）
4. Is [not] null,[not] like,[not] in
5. [not] between-and
6. 逻辑运算符: not
7. 逻辑运算符: and
8. 逻辑运算符: or

where 子句中的 **模糊查询** ：  
<code> like </code> ， like 匹配有 两个通配符：  
一个是 "**-**" 用于 "标识单个字符，只能是一个字符"。    
一个是 "**%**" 用于 "标识 0 到多个字符"。    

例：  
查找员工名字含有 字符e 的员工  
```sql
select * from emp
 where emp_name like '%e%';
```
查找 01 和 02 号部门中的所有员工  
```sql
select * from emp
 where emp.department_no in ('01','02');

--等价于
select * from emp
 where emp.department_no = '01' or emp.department_no='02';
```
<code> &gt;any </code> 演示：  
```sql
--工资 大于8000 的员
select * from emp
 where salary >all (4000,5000,8000);
```
** ALL  和  ANY ** :  
<code> &gt;ALL </code>，大于最大    
<code> &lt;ALL </code>，小于最小  
<code> &gt;ANY </code>，大于最小  
<code> &lt;ANY </code>，小于最大  

# 2 排序
查表 emp，按照 工资降序 排列：  
```sql
select *
  from emp
 where salary is not null
 order by salary desc;
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_195633.png)  

查表 emp，按照 工资降序 同时 入职日期从早到晚(从小到大) 排列：  
```sql
select *
  from emp
 where salary is not null
 order by salary desc, hire_date;
```
结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-22_195810.png)  

注意：  

1. **order by** 必须出现在 select 中的最后一句。  

2. **order by** 的排序条件是多列时，左边的排序优先级 **高于** 右边。   

# 3 聚合函数(分组函数)
聚合函数也叫分组函数。  
## 3.1 聚合函数(分组函数)
聚合函数(分组函数)的特点：  
1、 **多行数据参与运算返回一行结果**。  
2、 **主要用于统计**。  

五个聚合函数：  
1、 最值， **MAX** 和 **MIN**。可以统计任何数据类型， 包括数字，字符，日期。  
2、 平均值， **AVG**。聚合函数会忽略空值，注意！   
3、 求和，**SUM**  。
4、 统计记录条数，**count**。一般用 count(1) 或 count(\*)。  

示例：  
查询员工最晚(最大)入职日期  
```sql
select max(hire_date)
  from emp;
--结果：'2018-7-3'
```

查询员工平均工资  
```sql
select avg(nvl(salary,0))
  from emp;
--结果：'11040'
```
查询员工个数  
```sql
select count(1)
  from emp;
--结果：'10'
```

注意：聚合函数不能直接写在 where子句中。  

## 3.2 分组  

查看每个部门的薪水总和：按部门 **分组**，在对组内数据使用 **聚合函数(分组函数)** ，计算之后每组返回一条数据。  

```sql
select department_no,sum(salary) sum_salary
  from emp
 group by department_no
 order by sum_salary desc;
```
注意：  
1. 若 <code>group by</code> 中出现多列，那么会按照这几列的 **组合值相同的数据** 看做一组。  
2. 分组分出的小组有多少，结果集就有多少条。  
3. 只有在 <code>group by</code> 子句中出现的字段，才能 **直接** 写在 <code>select</code> 子句中。  

**分组** 常常和 **分组函数** 一起使用：  
```sql
--根据部门分组，计算每个部门的最高/低工资，平均工资，工资总和
select max(salary),min(salary),avg(salary),sum(salary)
  from emp
 group by department_no;
```
<code>having</code> 为 <code>group by</code> 添加过滤条件：  
```sql
select department_no, count(*)
  from emp
 group by department_no;
--结果：
--	01	4
--	02	6

select department_no, count(*)
  from emp
 group by department_no
having count(*)>4;
--结果：
--	02	6
```

# 4 提高查询效率

查询语句的执行顺序：  

1. from 子句。从右往左。  

2. where 子句。从右往左。**将能过滤掉最大数量记录的条件写在 where 子句的最右边**。   

3. group by。从左往右。**最好在 group by 之前使用 where 将不需要的记录在 group by 之前过滤掉**。    

4. having。**消耗资源，尽量避免使用**。  

5. select。 **尽量不要使用 星号(\*)**。  

6. order by。从左往右排序。  
