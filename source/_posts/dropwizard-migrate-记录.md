---
title: dropwizard migrate 记录
date: 2020-03-26 23:47:09
tags: dropwizard
---
# 1 Dropwizard Migrations

参考 [官网](https://dropwizard.readthedocs.io/en/stable/manual/migrations.html).

```sh
# 正常添加字段: 先在 xml 中加 changeSet, 再执行下面命令
java -jar hello-world.jar db migrate helloworld.yml
```

# 2 手动在数据添加

如果手动写 sql 做 DDL 操作, 需要在 migrations.xml (和 migrations_for_unittests.xml)
中 添加 相应的 DDL 操作.  

比如, 在 products 表中 手动加了一个字段 test_column:  
```sql
ALTER TABLE products ADD COLUMN test_column text ;
```

需要在 products_table.xml(被 migrations.xml 和 migrations_for_unittests.xml 引用) 中加入:
```xml
<changeSet id="add_test_column" author="cx">
        <sql>ALTER TABLE products ADD COLUMN test_column text</sql>
</changeSet>
```

然后运行:
```sh
# 注意如果直接运行 java -jar hello-world.jar db migrate helloworld.yml
# 会报错: test_column 已经存在

# fast-forward changeSet, 只在 databasechangelog 记录, 不会再做 DDL
java -jar hello-world.jar db fast-forward -i add_test_column helloworld.yml

# 此时, 再运行 java -jar hello-world.jar db migrate helloworld.yml 就正常了
```

到数据库查看 databasechangelog :  
```sql
select * from databasechangelog order by dateexecuted desc limit 10;
```

# 3 总结
如果是一张很大的表, 执行 db migrate 可能会长时间锁表, 应该考虑手动添加 再 db fast-forward.  
(待验证)  

两个未解决的问题:  
学习 Liquibase 如何工作的?  
如何 unit test DB ?
(三周内更新)
