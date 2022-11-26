---
title: 网络虚拟化之Linux Bridge和vlan
tags:
  - OpenStack
  - Neutron
comments: true
toc: true
mathjax: true
abbrlink: f8453b25
date: 2020-05-09 20:05:32
urlname:
categories:
thumbnail:
---



网络虚拟化中最重要的两个东西：Linux Bridge 和 VLAN

## Linux Bridge 基本概念

假设宿主机有 1 块与外网连接的物理网卡 eth0，上面跑了 1 个虚机 VM1，现在有个问题是：
如何让 VM1 能够访问外网？

至少有两种方案

1. 将物理网卡eth0直接分配给VM1，但随之带来的问题很多：
   宿主机就没有网卡，无法访问了；
   新的虚机，比如 VM2 也没有网卡。
   下面看推荐的方案
2. 给 VM1 分配一个虚拟网卡 vnet0，通过 Linux Bridge br0 将 eth0 和 vnet0 连接起来，如下图所示
   

![](https://pic.downk.cc/item/5eb69d4bc2a9a83be502b076.png)



Linux Bridge 是 Linux 上用来做 TCP/IP 二层协议交换的设备，其功能大家可以简单的理解为是一个二层交换机或者 Hub。多个网络设备可以连接到同一个 Linux Bridge，当某个设备收到数据包时，Linux Bridge 会将数据转发给其他设备。

在上面这个例子中，当有数据到达 eth0 时，br0 会将数据转发给 vnet0，这样 VM1 就能接收到来自外网的数据；
反过来，VM1 发送数据给 vnet0，br0 也会将数据转发到 eth0，从而实现了 VM1 与外网的通信。

现在我们增加一个虚机 VM2，如下图所示

![](https://pic.downk.cc/item/5eb69d97c2a9a83be503271f.png)



VM2 的虚拟网卡 vnet1 也连接到了 br0 上。
现在 VM1 和 VM2 之间可以通信，同时 VM1 和 VM2 也都可以与外网通信。





-------

有两点需要注意：
1. 之前宿主机的 IP 是通过 dhcp 配置在 eth0 上的；创建 Linux Bridge 之后，IP 就必须放到 br0 上了
2. 在 br0 的配置信息中请注意最后一行 “bridge_ports eth0”，其作用就是将 eth0 挂到 br0 上

重启宿主机，查看 IP 配置，可以看到 IP 已经放到 br0 上了

```# ifconfig  br0       Link encap:Ethernet  HWaddr 00:0c:29:8d:ec:be
          inet addr:192.168.111.107  Bcast:192.168.111.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe8d:ecbe/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:22573 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2757 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:4927580 (4.9 MB)  TX bytes:368895 (368.8 KB)

eth0      Link encap:Ethernet  HWaddr 00:0c:29:8d:ec:be
          inet6 addr: fe80::20c:29ff:fe8d:ecbe/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:24388 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2816 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:5590438 (5.5 MB)  TX bytes:411558 (411.5 KB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:146 errors:0 dropped:0 overruns:0 frame:0
          TX packets:146 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:10521 (10.5 KB)  TX bytes:10521 (10.5 KB)

virbr0    Link encap:Ethernet  HWaddr 72:db:fb:f2:19:91
          inet addr:192.168.122.1  Bcast:192.168.122.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

用 brctl show 查看当前 Linux Bridge 的配置。
eth0 已经挂到 br0 上了

```# brctl show 
bridge name     bridge id          STP enabled    interfaces
br0        8000.000c298decbe       no                    eth0
virbr0     8000.000000000000       yes
```

除了 br0，大家应该注意到还有一个 virbr0 的 Bridge，而且 virbr0 上已经配置了 IP 地址 192.168.122.1。
virbr0 的作用在后面介绍。



启动 VM1，看会发生什么

``` # virsh start VM1
 Domain VM1 started

 # brctl show
 bridge name     bridge id               STP enabled   interfaces
 br0             8000.000c298decbe       no                  eth0
                                                            vnet0
 virbr0          8000.000000000000       yes
```

brctl show 告诉我们 br0 下面添加了一个 vnet0 设备，通过 virsh 确认这就是VM1的虚拟网卡。

```# virsh domiflist 
VM1 Interface  Type       Source     Model       MAC 
------------------------------------------------------- 
vnet0      bridge     br0        rtl8139     52:54:00:75:dd:1a
```

VM1 的 IP 是 DHCP 获得的（设置静态 IP 当然也可以），通过 virt-manager 控制台登录 VM1，查看 IP。

```# ifconfig eth0      Link encap:Ethernet  HWaddr 52:54:00:75:dd:1a
          inet addr:192.168.111.106  Bcast:192.168.111.255  Mask:255.255.255.0
          inet6 addr: fe80::5054:ff:fe75:dd1a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:400 errors:0 dropped:0 overruns:0 frame:0
          TX packets:101 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:41950 (41.9 KB)  TX bytes:12583 (12.5 KB)
```

VM1 通过 DHCP 拿到的 IP 是 192.168.111.106，与宿主机（IP为192.168.111.107）是同一个网段。Ping 一下外网

```root@VM1:~# ping www.baidu.com 
PING www.a.shifen.com (180.97.33.108) 56(84) bytes of data. 
64 bytes from 180.97.33.108: icmp_seq=1 ttl=53 time=34.9 ms
64 bytes from 180.97.33.108: icmp_seq=2 ttl=53 time=36.2 ms
64 bytes from 180.97.33.108: icmp_seq=3 ttl=53 time=38.8 ms
```

---



## virbr0

virbr0 是 KVM 默认创建的一个 Bridge，其作用是为连接其上的虚机网卡提供 NAT 访问外网的功能。

<font size=2 color=red>**NAT**是网络地址穿透技术，内部网段的ip地址共用一个公共ip地址，内部发送报文的头部都由NAT网关自动替换为公共ip，接受报文同理</font>

也就是说一个物理主机上建立的多个虚机相当于一个局域网，不可能为每个虚拟网卡都分配公共ip，则通过virbr0充当NAT网关的作用，对外共用一个ip地址。

virbr0 默认分配了一个IP 192.168.122.1，并为连接其上的其他虚拟网卡提供 DHCP 服务。

在 virt-manager 打开 VM1 的配置界面，网卡 Source device 选择 “default”，将 VM1 的网卡挂在 virbr0 上。

启动 VM1，brctl show 可以查看到 vnet0 已经挂在了 virbr0 上。

```# brctl show
bridge name bridge id STP enabled interfaces
br0 8000.000c298decbe no eth0
virbr0 8000.fe540075dd1a yes vnet0
```

用 virsh 命令确认 vnet 就是 VM1 的虚拟网卡。

```
# virsh domiflist VM1

Interface Type Source Model MAC
——————————————————-
vnet0 network default rtl8139 52:54:00:75:dd:1a
```

virbr0 使用 dnsmasq 提供 DHCP 服务，可以在宿主机中查看该进程信息

```
# ps -elf|grep dnsmasq

5 S libvirt+ 2422 1 0 80 0 - 7054 poll_s 11:26 ? 00:00:00 /usr/sbin/dnsmasq –conf-file=/var/lib/libvirt/dnsmasq/default.conf
```

在 /var/lib/libvirt/dnsmasq/ 目录下有一个 default.leases 文件，当 VM1 成功获得 DHCP 的 IP 后，可以在该文件中查看到相应的信息

```
# cat /var/lib/libvirt/dnsmasq/default.leases

1441525677 52:54:00:75:dd:1a 192.168.122.6 ubuntu *
```

上面显示 192.168.122.6 已经分配给 MAC 地址为 52:54:00:75:dd:1a 的网卡，这正是 vnet0 的 MAC。之后就可以使用该 IP 访问 VM1 了。

```
# ssh 192.168.122.6

root@192.168.122.6’s password:
Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-30-generic x86_64)
Last login: Sun Sep 6 01:30:23 2015
root@VM1:~# ifconfig
eth0 Link encap:Ethernet HWaddr 52:54:00:75:dd:1a
inet addr:192.168.122.6 Bcast:192.168.122.255 Mask:255.255.255.0
inet6 addr: fe80::5054:ff:fe75:dd1a/64 Scope:Link
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:61 errors:0 dropped:0 overruns:0 frame:0
TX packets:66 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:7453 (7.4 KB) TX bytes:8649 (8.6 KB)

Ping一下外网。

root@VM1:~# ping www.baidu.com
PING www.a.shifen.com (180.97.33.107) 56(84) bytes of data.
64 bytes from 180.97.33.107: icmp_seq=1 ttl=52 time=36.9 ms
64 bytes from 180.97.33.107: icmp_seq=2 ttl=52 time=119 ms
64 bytes from 180.97.33.107: icmp_seq=3 ttl=52 time=88.5 ms
64 bytes from 180.97.33.107: icmp_seq=4 ttl=52 time=38.0 ms
64 bytes from 180.97.33.107: icmp_seq=5 ttl=52 time=122 ms
```

没有问题，可以访问外网，说明 NAT 起作用了。

需要说明的是，使用 NAT 的虚机 VM1 可以访问外网，但外网无法直接访问 VM1。
因为 VM1 发出的网络包源地址并不是 192.168.122.6，而是被 NAT 替换为宿主机的 IP 地址了。

这个与使用 br0 不一样，在 br0 的情况下，VM1 通过自己的 IP 直接与外网通信，不会经过 NAT 地址转换。

------

## VLAN

LAN 表示 Local Area Network，本地局域网，通常使用 Hub 和 Switch 来连接 LAN 中的计算机。一般来说，两台计算机连入同一个 Hub 或者 Switch 时，它们就在同一个 LAN 中。

一个 LAN 表示一个广播域。
其含义是：LAN 中的所有成员都会收到任意一个成员发出的广播包。

VLAN 表示 Virtual LAN。
一个带有 VLAN 功能的switch 能够将自己的端口划分出多个 LAN。
计算机发出的广播包可以被同一个 LAN 中其他计算机收到，但位于其他 LAN 的计算机则无法收到。
简单地说，VLAN 将一个交换机分成了多个交换机，限制了广播的范围，在二层将计算机隔离到不同的 VLAN 中。

比方说，有两组机器，Group A 和 B。
我们想配置成 Group A 中的机器可以相互访问，Group B 中的机器也可以相互访问，但是 A 和 B 中的机器无法互相访问。
一种方法是使用两个交换机，A 和 B 分别接到一个交换机。
另一种方法是使用一个带 VLAN 功能的交换机，将 A 和 B 的机器分别放到不同的 VLAN 中。

请注意：
VLAN 的隔离是二层上的隔离，A 和 B 无法相互访问指的是二层广播包（比如 arp）无法跨越 VLAN 的边界。
但在三层（比如IP）上是可以通过路由器让 A 和 B 互通的。概念上一定要分清。

现在的交换机几乎都是支持 VLAN 的。
通常交换机的端口有两种配置模式： Access 和 Trunk。看下图
![](https://pic.downk.cc/item/5eb6a970c2a9a83be5145717.png)





Access 口
这些端口被打上了 VLAN 的标签，表明该端口属于哪个 VLAN。
不同 VLAN 用 VLAN ID 来区分，VLAN ID 的 范围是 1-4096。
Access 口都是直接与计算机网卡相连的，这样从该网卡出来的数据包流入 Access 口后就被打上了所在 VLAN 的标签。
Access 口只能属于一个 VLAN。

Trunk 口
假设有两个交换机 A 和 B。
A 上有 VLAN1（红）、VLAN2（黄）、VLAN3（蓝）；B 上也有 VLAN1、2、3
那如何让 AB 上相同 VLAN 之间能够通信呢？

办法是将 A 和 B 连起来，而且连接 A 和 B 的端口要允许 VLAN1、2、3 三个 VLAN 的数据都能够通过。这样的端口就是Trunk口了。
VLAN1、2、3 的数据包在通过 Trunk 口到达对方交换机的过程中始终带着自己的 VLAN 标签。

了解了 VLAN 的概念之后，我们来看 KVM 虚拟化环境下是如何实现 VLAN 的。还是先看图，

![](https://pic.downk.cc/item/5eb6a9cac2a9a83be514ecf7.png)



eth0 是宿主机上的物理网卡，有一个命名为 eth0.10 的子设备与之相连。
eth0.10 就是 VLAN 设备了，其 VLAN ID 就是 VLAN 10。
eth0.10 挂在命名为 brvlan10 的 Linux Bridge 上，虚机 VM1 的虚拟网卡 vent0 也挂在 brvlan10 上。

这样的配置其效果就是：
宿主机用软件实现了一个交换机（当然是虚拟的），上面定义了一个 VLAN10。
eth0.10，brvlan10 和 vnet0 都分别接到 VLAN10 的 Access口上。
而 eth0 就是一个 Trunk 口。
VM1 通过 vnet0 发出来的数据包会被打上 VLAN10 的标签。

eth0.10 的作用是：定义了 VLAN10
brvlan10 的作用是：Bridge 上的其他网络设备自动加入到 VLAN10 中

我们再增加一个 VLAN20，见下图

这样虚拟交换机就有两个 VLAN 了，VM1 和 VM2 分别属于 VLAN10 和 VLAN20。
对于新创建的虚机，只需要将其虚拟网卡放入相应的 Bridge，就能控制其所属的 VLAN。

VLAN 设备总是以母子关系出现，母子设备之间是一对多的关系。
一个母设备（eth0）可以有多个子设备（eth0.10，eth0.20 ……），而一个子设备只有一个母设备。
