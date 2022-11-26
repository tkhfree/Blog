---
title: Linux里export和source的作用
tags:
  - Linux
comments: true
toc: true
mathjax: true
abbrlink: 3e4ce446
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---

export定义了之后变量变为系统变量，可以再本进程和子进程中调用，但是在脚本中定义了之后，在登录的shell进程中不能调用，因为登录的shell进程是该脚本的父进程
source可以免除注销再登录的变量定义，例如在脚本中定义了变量，source该脚本，则在登录的shell进程中可以调用，相当于没有再执行子进程

**总结：export定义的变量可以在本进程和子进程中使用，不能在父进程中调用；source脚本是把脚本在登录shell中执行，没有开子进程**

**linux的网络配置主要在/etc里**

/etc/sysconfig/network-scrips/ifcfg-en0
设定网卡的参数，例如ip、子网掩码、路由器、IP获取方式等

/etc/hosts
设定主机别名和ip的对应

/etc/services
设置Linux里服务进程的端口

/etc/resolve.conf
设置dns的地址


# Linux的网络指令
ifconfig 查看网卡信息

route 查看路由表

ping 测试通不通

traceroute 查看路由转发的节点

netstat 查看网络服务的端口启动情况 netstat -tulnp

host 查看域名对应的ip，调用/etc/resolv.conf的dns服务器查询

telnet ftp ssh wget

## tcpdump 封包分析指令
[root@linux ~]# tcpdump [-nn] [-i 介面] [-w 储存档名] [-c 次数] [-Ae]
                        [-qX] [-r 档案] [所欲撷取的资料内容]
参数：
-nn：直接以 IP 及 port number 显示，而非主机名与服务名称
-i ：后面接要‘监听’的网路介面，例如 eth0, lo, ppp0 等等的介面；
-w ：如果你要将监听所得的封包资料储存下来，用这个参数就对了！后面接档名
-c ：监听的封包数，如果没有这个参数， tcpdump 会持续不断的监听，
     直到使用者输入 [ctrl]-c 为止。
-A ：封包的内容以 ASCII 显示，通常用来捉取 WWW 的网页封包资料。
-e ：使用资料连接层 (OSI 第二层) 的 MAC 封包资料来显示；
-q ：仅列出较为简短的封包资讯，每一行的内容比较精简
-X ：可以列出十六进位 (hex) 以及 ASCII 的封包内容，对于监听封包内容很有用
-r ：从后面接的档案将封包资料读出来。那个‘档案’是已经存在的档案，
     并且这个‘档案’是由 -w 所制作出来的。
所欲撷取的资料内容：我们可以专门针对某些通讯协定或者是 IP 来源进行封包撷取，
     那就可以简化输出的结果，并取得最有用的资讯。常见的表示方法有：
     'host foo', 'host 127.0.0.1' ：针对单部主机来进行封包撷取
     'net 192.168' ：针对某个网域来进行封包的撷取；
     'src host 127.0.0.1' 'dst net 192.168'：同时加上来源(src)或目标(dst)限制
     'tcp port 21'：还可以针对通讯协定侦测，如 tcp, udp, arp, ether 等
     还可以利用 and 与 or 来进行封包资料的整合显示呢！



nc 或者 netcat 
nc -l -p 20000 启用tcp协议的20000端口listen
nc localhost 20000 连接20000端口
可以在两个bash中进行同步通信


