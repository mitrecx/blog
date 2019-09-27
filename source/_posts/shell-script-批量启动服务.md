---
title: 'shell script: 批量启动服务'
date: 2019-09-27 12:08:15
tags: shell
categories: shell
---

目标:  
写一个 shell 脚本, 启动多个 worker 服务.  
(worker 服务 用于处理 消息中间件 里的消息)  

先了解一些基础知识, 然后利用这些基础知识 [完成目标](#multi-start).  

# 1 读取命令行参数

|变量 |	含义|
|-- | -- |
|$0 |	当前脚本的文件名|
|$n |	传递给脚本或函数的参数. 第一个参数是 $1，第二个参数是 $2|
|$# |	传递给脚本或函数的 参数个数|
|$* |	传递给脚本或函数的 **所有参数** |
|$@ |	传递给脚本或函数的 **所有参数** |
|$? |	上个命令的退出状态，或函数的返回值。|
|$$ |	当前Shell进程ID|

注意 $* 和 $@ 的区别:  
```sh
$* 获得到的参数是 数组类型, "$*" 获得的参数是 字符串类型

$@ 和 "$@" 获得的参数 都是 数组类型
```
示例代码 **t2.sh** :  
```sh
echo ---"print each param from \$*"
for var in $*
do
    echo "$var"
done
echo ---"print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done

echo ---"print each param from \$@"
for var in $@
do
    echo "$var"
done
echo ---"print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```
<code> ./t2.sh mitre cherry</code>  执行结果:  
```
---print each param from $*
mitre
cherry
---print each param from "$*"
mitre cherry
---print each param from $@
mitre
cherry
---print each param from "$@"
mitre
cherry
```

# 2 Shell 基本运算
Shell 支持多种运算符，包括：  
1, 算数运算符  
2, 关系运算符  
3, 逻辑运算符
4, 字符串运算符(略)  
5, 文件测试运算符(略)  

## 2.1 算数运算
原生 bash 不支持简单的数学运算，但是可以通过其他命令来实现，比如 expr 或 awk.  

expr 是一款表达式计算工具，使用它能完成表达式的求值操作.  
```sh
#!/bin/bash
val=`expr 2 + 2`
echo "两数之和为 : $val"
```
运行结果:  
```
两数之和为 : 4
```
**<font color=red>注意: </font>**  
1, 表达式 和 **<font color='#8a00e6'>运算符</font>** 之间要有空格，例如 **2<font color='#8a00e6'>+</font>2** 是不对的，必须写成 **2 <font color='#8a00e6'>+</font> 2**，这与我们熟悉的大多数编程语言不一样.  
2, 完整的 表达式 要被 \` \` 包含.  

运算符	|说明|	举例
--|--|--
+|	加|\`expr $a + $b\` 结果为 30。
\-	|减	|\`expr $a - $b\` 结果为 -10。
\*	|乘	|\`expr $a \* $b\` 结果为  200。
/	|除	|\`expr $b / $a\` 结果为 2。
%	|取余	|\`expr $b % $a\` 结果为 0。

## 2.2 关系运算  
**关系运算符** 只支持 **数字** ，不支持字符串，除非字符串的值是数字.  

假定变量 a 为 10，变量 b 为 20:  

运算符|说明|	举例  
--|--|--
-eq|	equal |	[ $a -eq $b ] 返回 false
-ne|	not equal |	[ $a -ne $b ] 返回 true
-gt|	greater than |[ $a -gt $b ] 返回 false
-lt|	less than | [ $a -lt $b ] 返回 true
-ge|	greater than or equal | [ $a -ge $b ] 返回 false
-le|	less than or equal | [ $a -le $b ] 返回 true


## 2.3 逻辑运算

假定变量 a 为 10，变量 b 为 20:

运算符|	说明	|举例
--|--|--
&&|	与 AND|	[[ $a -lt 123 && $b -gt 123 ]] 返回 false
&#124;&#124; |	或 OR |	[[ $a -lt 123 &#124;&#124; $b -gt 123 ]] 返回 true
! | 非 | 见下例

例子:  
```sh
if [[ ! 8 -eq 9 ]] ; then
  echo "8 not equal 9"
else
  echo "8 equal 9"
fi
```
运行结果:  
```
8 not equal 9
```


<h1 id="multi-start">3 多服务启动脚本</h1>

了解前面的基础知识, 写 一个启动多个服务的脚本 很简单了:  

```sh
#!/bin/bash

java_dir="/app/jdk1.8.0_191/bin/java"
start_port=4242
worker_cnt=1
jar_name="worker-0.0.1-SNAPSHOT.jar"

usage()
{
  (echo "使用方法: $0 起始端口号 worker个数"
   echo "起始端口号 必须大于 0"
   echo "worke 个数必须大于 0 同时小于16"
   ) 1>&2
}
# 限制参数输入
if [ $# -ne 2 ]; then
  usage
  exit 1
fi

# 判断起始端口号是否 合法
if [ $1 -gt 0 ] 2>/dev/null ;then  
  start_port=$1
else  
  usage
  exit 1
fi   
# worker 个数必须是大于0的整数
if [[ $2 -gt 0 && $2 -lt 16 ]] 2>/dev/null ;then  
  worker_cnt=$2
else  
  usage
  exit 1
fi   
# 删除日志文件
rm nohup*.out

echo "正在启动..."
for ((i=0;i<$worker_cnt;i++)); do
  #echo `expr $start_port + $i`
  port=`expr $start_port + $i`
  echo "nohup "${java_dir}" -Xms1024m -Xmx1024m -jar ${jar_name} --server.port=${port} > nohup${port}.out 2>&1 &"
  nohup "${java_dir}" -Xms1024m -Xmx1024m -jar ${jar_name} --server.port=${port} > nohup${port}.out 2>&1 &  
  sleep 4
done
echo "启动完成!"
```
**说明:**  
<code>1>&2</code> 把 **标准输出** 重定向到 **标准错误** .  
因为 标准输出 和 标准错误 最终都是输出到 终端.  
所以加或不加 <code>1>&2</code>, 运行结果是一样的 .

<code>2>/dev/null</code> 把 标准错误 重定向到 **空设备文件** 中.  

# 4 关闭多个进程  
一次性启动了多个进程, 一个一个的手动 kill, 挺费事的.  
不如 写一个脚本, 一次性关闭 所有 worker 进程:  

```sh
#!/bin/bash

ps -ef | grep worker-0.0.1-SNAPSHOT | awk '{print $2}' | xargs -I{}  kill -9 {}
```
因为 ps 进程 也会产生一个 pid, kill 的 时候会提示:  
**kill 27846: No such process**  
这是因为 ps 进程 在 执行 kill命令前就 已经 关闭了.

# References
1 [shell 命令行参数](https://www.cnblogs.com/kingstrong/p/6026896.html)  
2 [shell 基本运算符](https://www.runoob.com/linux/linux-shell-basic-operators.html)  
3 [start.sh](https://github.com/LARG/utaustinvilla3d/blob/master/start.sh)  
