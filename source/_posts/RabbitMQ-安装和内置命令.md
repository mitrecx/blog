---
title: RabbitMQ 安装和内置命令
date: 2019-09-17 18:40:52
tags: 消息中间件
categories: 消息中间件
---
# 1. RabbitMQ 背景

虽然在同步消息通讯的世界里有很多公开标准 (如 COBAR的 IIOP , 或者是 SOAP 等) .  
但是在异步消息处理中却不是这样, 只有大企业有一些商业实现 (如微软的 MSMQ , IBM 的 Websphere MQ 等), 在 2006 年的 6 月, Cisco 、Redhat、iMatix 等联合制定了 AMQP (Advanced Message Queue, 高级消息队列协议).  

RabbitMQ 是一个 用 erlang 语言 编写的 AMQP 的 **开源实现** .  

RabbitMQ 是由 RabbitMQ Technologies Ltd 开发并且提供商业支持.  
该公司在 2010年4月 被 SpringSource ( VMWare的一个部门 ) 收购.  
在2013年5月被并入 Pivotal (Spring 框架就是 Pivotal 开发的).  
其实 VMWare , Pivotal 和 EMC 本质上是一家的.  


# 2. 下载并安装 RabbitMQ
目前, RabbitMQ 最新的发布版本是 3.7.17 . 有关发行说明, 参见 [changelog](https://www.rabbitmq.com/changelog.html) .  
安装方法参见这个 [页面](https://www.rabbitmq.com/download.html) . 不管是 Linux, BSD, UNIX, 还是 Windows, 还是 MacOS 都有对应的 安装指南.  

因为工作在云内, 不能连公网的缘故, 所以我在 云内主机上(Redhat 6.7) 离线安装的 RabbitMQ .  

这里简单说明一下, 如何 在 RedHat 6.7 上离线安装 RabbitMQ:  
1, 下载 [erlang-22.0.7-1.el6.x86_64.rpm](https://packagecloud.io/rabbitmq/erlang).   
2, 下载 [socat-1.7.2.3-1.el6.x86_64.rpm](http://www.rpmfind.net/linux/rpm2html/search.php?query=socat(x86-64)), el 表示Red Hat **E** nterprise **L** inux, EL6 是 Red Hat 6.x.  
3, 下载[rabbitmq-server-3.7.17-1.el6.noarch.rpm](https://www.rabbitmq.com/install-rpm.html#downloads).  
4, 用 root 用户 执行如下命令:  
```sh
sudo rpm -ivh erlang-22.0.7-1.el6.x86_64.rpm
sudo rpm -ivh socat-1.7.2.3-1.el6.x86_64.rpm
sudo rpm -ivh rabbitmq-server-3.7.17-1.el6.noarch.rpm
```
5, 启动 RabbitMQ:  
```sh
service rabbitmq-server start
```

# 2. RabbitMQ 基本概念  
**Broker**: 简单来说就是消息队列服务器实体(代理).  

Exchange: 消息交换机，它指定消息按什么规则，路由到哪个队列.  

**Queue**: 消息队列载体，每个消息都会被投入到一个或多个队列.  

Binding: 绑定，它的作用就是把exchange和queue按照路由规则绑定起来.  

**Routing Key**: 路由关键字，exchange根据这个关键字进行消息投递.  

vhost: 虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离.  

**channel**: 消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务.  

producer: 消息生产者，就是投递消息的程序.  

consumer: 消息消费者，就是接受消息的程序.  

# 3. RabbitMQ 常用命令  

## 3.1 服务启动与关闭
```sh
# 启动
service rabbitmq-server start
# 关闭
service rabbitmq-server stop
# 重启
service rabbitmq-server restart
```

## 3.2 RabbitMQ 提供的 命令行工具

RabbitMQ comes with multiple **Command Line Tools**, Different tools cover different usage scenarios :

**<font color='#8a00e6'>rabbitmqctl</font>** for service management and general operator tasks.  
**<font color='#ff9900'>rabbitmq-diagnostics</font>** for diagnostics and health checking.  
**<font color='#8a00e6'>rabbitmq-plugins</font>**  for plugin management.  
**<font color='#ff9900'>rabbitmqadmin</font>**  for operator tasks over HTTP API.  

以上 4 个 命令行工具 都可以用 help 参数 查看使用方法, 例如:  
```sh
# 查看 rabbitmqctl 帮助
rabbitmqctl help
# 查看 rabbitmqctl 添加用户帮助
rabbitmqctl help add_user

# 查看 rabbitmq-plugins 帮助
rabbitmq-plugins help
```

静下心来把 帮助文档 读一遍, 同时把 [官网说明](https://www.rabbitmq.com/cli.html) 读一遍, 基本上所有命令就都能理解了.  



这里现在简单看下 **<font color='#8a00e6'>rabbitmqctl</font>** 和 **<font color='#8a00e6'>rabbitmq-plugins</font>** 命令行工具 提供的的常用命令 .  

**<font color='#8a00e6'>rabbitmqctl</font>** 是 RabbitMQ 附带的原生的 CLI 工具(CLI tool).  
**<font color='#8a00e6'>rabbitmqctl</font>** 主要提供一些 管理操作:  
* Stopping node  
* Access to node status, effective configuration, health checks  
* **Virtual host management**  
* User and permission management  
* Policy management  
* Listing queues, connections, channels, exchanges, consumers  
* **Cluster membership management**  

```sh
# 用户列表
rabbitmqctl list_users
# 新增
rabbitmqctl add_user mitre mitre
# 删除
rabbitmqctl delete_user mitre

# 设置角色
rabbitmqctl set_user_tags mitre administrator
# 设置用户权限 和 vhost
rabbitmqctl  set_permissions  [--vhost <vhost>] <username> <conf> <write> <read>
rabbitmqctl  set_permissions  --vhost / mitre . . .
```

**<font color='#8a00e6'>rabbitmq-plugins</font>** is a tool that **manages plugins**: lists, enables and disables them.  

```sh
# 列出所有 插件
rabbitmq-plugins list
# 启用 web 页面管理插件
rabbitmq-plugins enable rabbitmq_management
```

# References  
1 [Downloading and Installing RabbitMQ](https://www.rabbitmq.com/download.html)  
2 [Centos7 离线安装RabbitMQ,并配置集群](https://blog.csdn.net/Alger_magic/article/details/82868267)  
3 [RabbitMQ基础知识](https://www.cnblogs.com/dwlsxj/p/RabbitMQ.html)  
