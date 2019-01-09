---
title: Linux 解析文件并插入 Oracle 数据库
date: 2019-01-08 16:02:41
tags: ["Linux", "数据库"]
categories: Linux
---

# 1 场景

1 台 阿里的云主机 **Ubuntu16.04**， 安装 **Oracle 数据库 客户端** 12c R2。  
1 台 华为的云主机 **CentOS7**， 安装 **Oracle 数据库** 12c R2。  

目标： Ubuntu 解析文本文件，并 利用sqlplus 插入 远程Oracle数据库中。   

文本文件 batch.dat:  
```
1005|cherry|48|F|18
1007|jason|48|M|18
1008|嘉宣|20|F|1
1008|嘉文|20|F|1
```
数据库表 students 表结构：  
```
SQL> desc students;

Name     Type         Nullable Default Comments
-------- ------------ -------- ------- --------
STU_ID   VARCHAR2(4)  Y                         
STU_NAME VARCHAR2(50) Y                         
AGE      NUMBER(3)    Y                         
SEX      CHAR(1)      Y                         
GRADE    NUMBER(2)    Y                         
```

# 2 实现
tinsert.sh 文件：    
```sh
#!/bin/bash
./tparse.sh;

sqlplus -S system/123@114.115.165.64:1521/orcl<<EOF
@sql2.txt
commit;
exit
EOF
if [[ $? -eq 0 ]]; then
    echo "successfully!"
fi
```

tparse.sh 文件：   
```sh
#!/bin/bash
iconv -f gbk -t utf-8 batch.dat > tbatch.dat;
count=0;
# 在最后一行添加一行空行, 解决最后一行读取不到的问题
sed -i '$a\ ' tbatch.dat;
# 删除所有空行
sed -i '/^\s*$/d' tbatch.dat;

while read line
do
    echo $line;
    query=`echo $line | awk -F"|" '{
    printf("'\''%s'\'','\''%s'\'','\''%s'\'','\''%s'\'','\''%s'\''",$1,$2,$3,$4,$5);}'`;
    statement=`echo "insert into students(stu_id,stu_name,age,sex,grade) values($query);"`;
    if [[ count -eq 0 ]]; then
        echo $statement > sql.txt;
    else
        echo $statement >> sql.txt;
    fi  
    count=$((count+1));
    #echo $count;
    #echo $statement;
done < tbatch.dat
cat sql.txt > sql2.txt
# 去除Windows中的换行符 ^M
# cat -A sql.txt | sed 's/\^M//g' > sql2.txt
# sed -i 's/\$//g' sql2.txt
```

执行：  
```sh
./tinsert.sh
```

Windows 打开 PLSQL 连接 远程数据库，查询插入结果：  
![](https://mitre.oss-cn-hangzhou.aliyuncs.com/blog-2018-12/2019-01-08_162224.png)  
