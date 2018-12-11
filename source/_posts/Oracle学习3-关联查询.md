---
title: Oracle-学习3-关联查询
date: 2018-11-24 17:55:00
tags: ["数据库","Oracle"]
categories: 数据库
---
Oracle数据库 是关系型数据库。表与表之间可以存在关系(连接条件)。  

Oracle数据库的关联查询有两种：内连接(等值连接、自然连接) 和 外连接(左外，右外，全外)。  

先建立两张表如下：  
支用申请信息表  
```sql
create table loan_app_info(
app_no varchar2(25) not null,
channel_type varchar2(2),
loan_no varchar2(30)
);
comment on table loan_app_info
is '支用申请信息表';
comment on column loan_app_info.app_no
is '申请流水号';
comment on column loan_app_info.channel_type
is '渠道类型';
comment on column loan_app_info.loan_no
is '网贷生成出账号';
```
网贷借据表  
```sql
create table acc_loan(
bill_no varchar(30) not null,
contract_no varchar2(30),
prd_name varchar(30),
prd_type varchar(6),
cus_id varchar2(18),
cus_name varchar2(100)
);

comment on table acc_loan
is '网贷借据表';
comment on column acc_loan.bill_no
is '借据编号';
comment on column acc_loan.contract_no
is '合同编号';
comment on column acc_loan.prd_name
is '产品名称';
comment on column acc_loan.prd_type
is '产品类型';
comment on column acc_loan.cus_id
is '客户号';
comment on column acc_loan.cus_name
is '客户名称';
```
两张表存在 **关联关系**：loan_app_info.loan_no 与 acc_loan.bill_no (这两个字段来源相同)。  

插入如下数据：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/loan_app_info.png)  

![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/acc_loan.png)  

loan_app_info有 **9** 条记录，acc_loan有 **8** 条记录。两张表里能关联上的记录有 **6** 条。  

两表的 **笛卡尔积** 会有 72(9*8) 条记录:  
```sql
--笛卡尔积
select *
from loan_app_info,acc_loan ;
```
注：笛卡尔积 通常是对数据查询危害极大的操作。

# 1 内连接
内连接有两种： 等值连接 和 自然连接。  

## 1.1 等值连接
```sql
--内连接-等值连接1
select *
from loan_app_info app, acc_loan acc
where app.loan_no = acc.bill_no;
--结果：6条记录
```

```sql
--内连接-等值连接2
select *
from loan_app_info app inner join acc_loan acc on(app.loan_no=acc.bill_no);
--结果：6条记录
```
<code>"inner"</code> 和 <code>"()"</code>均可以省略。    

## 1.2 自然连接
自然连接 会自动寻找两张表中 **列名相同** 的列 做 **等值连接**。  
自然连接的条件：两张表 有且只有一列列名相同才能用自然连接。  

如果不满足上述条件，自然连接的结果是笛卡尔积。  

```sql
--内连接-自然连接
select *
from loan_app_info natural join acc_loan;
--因为列名不同，结果是笛卡尔积
```
我的建议： 不要使用自然连接，因为不够灵活。

# 2 外连接
外连接包括：左外连接，右外连接，全外连接。  

## 2.1 左外连接
以左边的表为主，即左边的表要全包含，右边表没有的记录可以补null。  
```sql
--外连接-左外连接
select *
from loan_app_info app left outer join acc_loan acc on(app.loan_no=acc.bill_no);
--结果：9条数据(不是6条)。左边表loan_app_info的数据全包含
```
<code>"outer"</code> 和 <code>"()"</code>均可以省略。  

结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-11/2018-11-26_185254.png)  

## 2.2 右外连接  
以右为主，左不全补null。  
```sql
--外连接-右外连接
select *
from loan_app_info app right outer join acc_loan acc on(app.loan_no=acc.bill_no);
--结果：8条数据，右边表数据全包含
```
<code>"outer"</code> 和 <code>"()"</code>均可以省略。  

## 2.3 全外连接
两边表的数据都要全包含。  
```sql
--外连接-全外连接
select *
from loan_app_info app full outer join acc_loan acc on(app.loan_no=acc.bill_no);
--结果：11条数据(6+ 9-6 + 8-6 =11)
```
<code>"outer"</code> 和 <code>"()"</code>均可以省略。  

# 3 自连接
自连接 一般用于 一张表内的 **字段间** 存在关联关系。  
