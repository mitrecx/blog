---
title: Linux 域名解析失败
date: 2019-01-14 15:00:23
tags: Linux
categories: Linux
---

# 1 问题
```sh
wget https://mirrors.163.com/centos/7/os/x86_64/Packages/PackageKit-yum-plugin-1.1.10-1.el7.centos.x86_64.rpm
```
报错如下：  
<font color=red>正在解析主机 mirrors.163.com... 失败：域名解析暂时失败。  
wget: 无法解析主机地址 “mirrors.163.com”</font>    

# 2 问题排查

```sh
# ping 百度的域名，ping不通
ping baidu.com
# ping: unknown host baidu.com

# ping 百度的 ip，可以 ping通
ping 220.181.112.244
# PING 220.181.112.244 (220.181.112.244) 56(84) bytes of data.
# 64 bytes from 220.181.112.244: icmp_seq=1 ttl=49 time=26.5 ms
# 64 bytes from 220.181.112.244: icmp_seq=2 ttl=49 time=26.3 ms
```
**说明是 DNS 解析 有问题！**  

修改 <font color='#0066ff'>/etc/resolv.conf</font> 文件(DNS resolver配置文件) ：  
```
nameserver 114.114.114.114
```
**114DNS** 以多个基础电信运营商自用的DNS系统为基础，通过扩展而建成专业的第三方高可靠 DNS服务平台....  
总之，用 114DNS 做 DNS解析 能解决当下的问题。  

结果：  
```sh
ping baidu.com
# PING baidu.com (220.181.57.216) 56(84) bytes of data.
# 64 bytes from 220.181.57.216: icmp_seq=1 ttl=49 time=22.3 ms
# 64 bytes from 220.181.57.216: icmp_seq=2 ttl=49 time=22.2 ms
```

# 3 附加说明  

Linux 版本信息：  
```sh
uname -a
# Linux ahtelecom-rhel6.4 2.6.32-358.el6.x86_64 #1 SMP Tue Jan 29 11:47:41 EST 2013 x86_64 x86_64 x86_64 GNU/Linux

cat /etc/redhat-release
# Red Hat Enterprise Linux Server release 6.4 (Santiago)
```

网络配置信息：  
```sh
# 查询网关
route -n
# Kernel IP routing table
# Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
# 169.254.169.254 10.189.2.1      255.255.255.255 UGH   0      0        0 eth0
# 10.189.2.0      0.0.0.0         255.255.255.0   U     1      0        0 eth0
# 0.0.0.0         10.189.2.1      0.0.0.0         UG    0      0        0 eth0

# 网卡配置文件
cat /etc/sysconfig/network-scripts/ifcfg-eth0
# DEVICE=eth0
# TYPE=Ethernet
# ONBOOT=yes
# BOOTPROTO=dhcp
# NM_CONTROLLED=yes

# 服务器所需网络配置信息(information about the desired network configuration on your server)
cat /etc/sysconfig/network
# NETWORKING=yes
# HOSTNAME=ahtelecom-rhel6.4
```
