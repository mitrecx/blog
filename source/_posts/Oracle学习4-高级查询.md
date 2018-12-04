---
title: Oracle-学习4-高级查询
date: 2018-11-25 13:45:54
tags: ["Oracle","数据库"]
categories: 数据库
---
Oracle 高级查询：  
* 子查询(不做说明)  
* [分页查询](#head1)
* [decode 函数](#head2)
* [排序函数](#head3)
* [高级分组](#head4)
* [集合](#head5)

<h1 id="head1">1 分页查询</h1>

分页查询的思想："勤拿少取"    

分页查询步骤：  
1、 排序(非必要)  
2、 编号  
3、 取范围  

例如，对 emp表 进行分页查询。   
emp表 有如下记录：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-27_142648.png)   

## 1.1 排序

按照 入职日期 排序：   

```sql
select *
  from emp
 order by hire_date
```

## 1.2 编号

```sql
--先排序，再编号
select rownum rn,e.*
  from (
     select *
     from emp
     order by hire_date) e
```

错误做法：  
```sql
--排序与编号同时进行
select rownum rn,emp.*
  from emp
 order by hire_date
```
错误做法的结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-27_143212.png)  
由上图可见，**rownum 不能和 order by 写在一个查询语句中**。

## 1.3 取范围
假设当前是第 n 页， 一页的条数为 pageSize， 则页数的范围：  
start: ( n-1 ) \* pageSize + 1  
end: pageSize * n

```sql
--pageSize 取 6
select * from (
       select rownum rn,e.*
       from(
           select *
           from emp
           order by hire_date) e)
 where rn between 1 and 6;

select * from (
       select rownum rn,e.*
       from(
           select *
           from emp
           order by hire_date) e)
 where rn between 7 and 12;
```

<h1 id='head2'>2 decode 函数</h1>

**DECODE函数 的作用和 java 中的 switch-case 一样**。  

计算员工的奖金：如果部门号为 01，奖金为 [薪水\*2]。如果部门号为 02，奖金为 [薪水\*10]。其他部门，奖金额等于薪水值。  
```sql
select emp.*,decode(department_no,
                                 '01',salary*2,
                                 '02',salary*10,
                                 salary) bonus
from emp;
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-27_223602.png)  

Oracle 还提供了 case-when-then-else-end 语句，功能和 DECODE 类似：  
```sql
select emp.*,
       case department_no
         when '01' then salary*2
         when '02' then salary*10
         else salary end
       bonus
from emp;

--
select app.*,
  case
    when app.channel_type='01' then '网上银行'
    when app.channel_type='02' then '手机银行'
    else '其他渠道'
  end
  渠道
from loan_app_info app
```
**DECODE 与CASE WHEN 的比较:**  
1. DECODE 只有Oracle 才有，其它数据库不支持。  
CASE WHEN的用法， Oracle、SQL Server、 MySQL 都支持。  
2. DECODE 只能用做相等判断，但是可以配合sign函数进行大于，小于，等于的判断。  
CASE when可用于 =，>=，<，<=，<>，is null，is not null 等的判断。  
3. DECODE 使用其来比较简洁，CASE 虽然复杂但更为灵活。  
4. 在decode中，null和null是相等的。  
在case when中，只能用is null来判断。  


<h1 id='head3'>3 排序函数</h1>

## 3.1 普通排序
对 emp表 按工资排序：  
```sql
select e.*, rownum as 序号
  from (select *
          from emp
          where salary is not null
         order by salary desc) e;
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-27_231815.png)  

## 3.2 组内排序
<code>row_number() over()</code> 函数， 对 emp表按 部门编号 分组，再组内排序：  
```SQL
--row_number 生成组内 唯一且连续的编号
select emp.*, row_number() over(partition by department_no order by salary desc) rk
from emp
where salary is not null;
```
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-27_232515.png)  

```sql
--rank over,s 组内不连续且不唯一的编号
select emp.*, rank() over(partition by department_no order by salary desc) rk
from emp
where salary is not null;

--dense_rank over, 组内连续但不唯一的编号
select emp.*, dense_rank() over(partition by department_no order by salary desc) rk
from emp
where salary is not null;
```

<h1 id='head4'>4 高级分组方法</h1>

高级分组方法：rollup,**grouuping sets**(重点), cube    

建立一张营业额表：  
```sql
create table SALES_TAB
(
  year_id     NUMBER not null,
  month_id    NUMBER not null,
  day_id      NUMBER not null,
  sales_value NUMBER(10,2) not null
)
```
插入 1000 条数据，数据是每天的营业额(每天可能有多笔营业收入):  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-28_091712.png)  

## 4.1 group by 分组

group by 是最基础，也是最重要的分组方法。  
计算每天的营业额：  
```sql
select year_id,month_id,day_id,sum(sales_value)
  from sales_tab
 group by year_id, month_id,day_id
 order by year_id,month_id,day_id;
```

## 4.2 group by rollup 分组
按 **年月日** 分组，再按 **年月** 分组，再按 **年** 分组，**总的一组**。(一共4个分组)  ：  
```sql
select year_id,month_id,day_id,sum(sales_value)
  from sales_tab
 group by rollup( year_id, month_id,day_id)
 order by year_id,month_id,day_id;
```
输出结果片段：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-28_094354.png)
注意：  
**rollup** 内部的字段顺序决定了 分组结果。  
例如 rollup(1,2,3), 分组结果：123， 12， 1，总表一组  


## 4.3  group by grouping sets 分组
**grouping sets** 很灵活，分组结果可以自己指定。  
按 **年月日** 分组，再按 **年** 分组：  
```sql
select year_id,month_id,day_id,sum(sales_value)
from sales_tab
group by
     grouping sets(( year_id, month_id，day_id),
                   (year_id)
     )
order by year_id,month_id,day_id;
```
## 4.4 group by cube 分组
<code>cube(年,月,日)</code> 分成如下 8 种组合：
年月日，年月，年，  
年日，月日，  
月，日，  
总。  
```sql
select year_id,month_id,day_id,sum(sales_value)
  from sales_tab
 group by cube( year_id, month_id,day_id)
 order by year_id,month_id,day_id;
```

<h1 id='head5'>5 集合</h1>

集合运算符：  
**union/union all** 并集   
**intersect** 交集   
**minus** 差集  

1. union求并集，公共部分只 **包含一次**  

2. union all求集并，公共部分 **包含二次**  

3. intersect求交集，只有公共部分  

4. minus求差集，例如集合A减集合B  
