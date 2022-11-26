---
title: Neutron了解二
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: bb276a87
date: 2020-05-12 09:40:21
urlname:
categories:
thumbnail:
---

Neutron的ML2 plugin包含**type driver** 和**mechanism driver**，type driver决定了建立的网络类型，包括local、flat、vlan、vxlan、gre；mechanism driver决定了实现的技术，例如使用Linux bridge和ovs等。一个抽象驱动，一个建模驱动。

local只有同机同网络才能建立通讯；

flat不同机同网络可以通讯，一个虚机连一个网卡；

举例：

​		建立flat网络时，建立一个网桥和一个dhcp（subnet那里选择dhcp的话），当建立实例挂载在这个flat网络上时，同时建立网桥，dhcp tap，instance tap。

<font size=2>tap：虚拟网络接口，用于连接虚机管理程序。个人理解一个tap/tun从二层/三层上定义一个虚拟网卡。</font> 



**DHCP**

dhcp通过在控制节点/网络节点的dhcp agent提供组件，默认通过dnsmasq程序实现dhcp功能。

![](https://pic.downk.cc/item/5eba022bc2a9a83be5f85f76.jpg)

在dhcp配置参数里有一个 --interface  ，指定提供 DHCP 服务的 interface。dnsmasq 会在该 interface 上监听 instance 的 DHCP 请求。

Neutron 通过 dnsmasq 提供 DHCP 服务，而 dnsmasq 如何独立的为每个 network 服务呢？

通过 **Linux Network Namespace** 隔离

在二层网络上，VLAN 可以将一个物理交换机分割成几个独立的虚拟交换机。类似地，在三层网络上，Linux network namespace 可以将一个物理三层网络分割成几个独立的虚拟三层网络。

每个 namespace 都有自己独立的网络栈，包括 route table，firewall rule，network interface device 等。

Neutron 通过 namespace 为每个 network 提供独立的 DHCP 和路由服务，从而允许租户创建重叠的网络。如果没有 namespace，网络就不能重叠，这样就失去了很多灵活性。每个 dnsmasq 进程都位于独立的 namespace, 命名为 qdhcp-<network id>

宿主机本身也有一个 namespace，叫 root namespace，拥有所有物理和虚拟 interface device。物理 interface 只能位于 root namespace。



新创建的 namespace 默认只有一个 loopback device。管理员可以将虚拟 interface，例如 bridge，tap 等设备添加到某个 namespace。



对于 flat_net 的 DHCP 设备 tap19a0ed3d-fe，需要将其放到 namespace qdhcp-f153b42f-c3a1-4b6c-8865-c09b5b2aa274 中，但这样会带来一个问题：tap19a0ed3d-fe 将无法直接与 root namespace 中的 bridge 设备 brqf153b42f-c3 连接。



**Neutron 使用 veth pair 解决了这个问题。**



veth pair 是一种成对出现的特殊网络设备，它们象一根虚拟的网线，可用于连接两个 namespace。向 veth pair 一端输入数据，在另一端就能读到此数据。



tap19a0ed3d-fe 与 ns-19a0ed3d-fe 就是一对 veth pair，它们将 qdhcp-f153b42f-c3a1-4b6c-8865-c09b5b2aa274 连接到 brqf153b42f-c3。

---

以 cirros-vm1 为例分析获取 DHCP IP 的详细过程。



在创建 instance 时，Neutron 会为其分配一个 port，里面包含了 MAC 和 IP 地址信息。这些信息会同步更新到 dnsmasq 的 host 文件。同时 nova-compute 会设置 cirros-vm1 VIF 的 MAC 地址。

一切准备就绪，instance 获取 IP 的过程如下：



1. cirros-vm1 开机启动，发出 DHCPDISCOVER 广播，该广播消息在整个 flat_net 中都可以被收到。



2. 广播到达 veth tap19a0ed3d-fe，然后传送给 veth pair 的另一端 ns-19a0ed3d-fe。dnsmasq 在它上面监听，dnsmasq 检查其 host 文件，发现有对应项，于是dnsmasq 以  DHCPOFFER 消息将 IP（172.16.1.103）、子网掩码（255.255.255.0）、地址租用期限等信息发送给 cirros-vm1。



3. cirros-vm1 发送 DHCPREQUEST 消息确认接受此 DHCPOFFER。



4. dnsmasq 发送确认消息 DHCPACK，整个过程结束。



这个过程我们可以在 dnsmasq 日志中查看。

dnsmasq 默认将日志记录到 /var/log/syslog。

---

**Routing 功能概述**

路由服务（Routing）提供跨 subnet 联通功能。例如前面我们搭建了[实验环境](http://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587298&idx=1&sn=7ccef4b40e40edea6f1d45b87930a42e&chksm=8d308f7bba47066db7bcc6ba93b86adc2c9ab0b16a897411dd5f3094c411019f0235d260a905&scene=21#wechat_redirect)：



cirros-vm1    172.16.100.3     vlan100

cirros-vm3    172.16.101.3     vlan101



这两个 instance 要通信必须借助 router。可以是物理 router 或者虚拟 router。

![](https://pic.downk.cc/item/5eba1c11c2a9a83be525dde5.png)



**为什么要把 router_100_101 放到 namespace 中**？

讨论一个深层次的问题：



为什么不直接在 tape17162c5-00 和 tapd568ba1a-74 上配置 Gateway IP，而是引入一个 namespace，在 namespace 里面配置 Gateway IP 呢？



首先考虑另外一个问题：



如果不用 namespace，直接 Gareway IP 配置到 tape17162c5-00 和 tapd568ba1a-74 上，能不能连通 subnet_172_16_100_0 和 subnet_172_16_101_0 呢？



答案是可以的，只要控制节点上配置了类似下面的路由。

> Destination Gateway Genmask Flags Metric Ref Iface
>
> 172.16.100.0 * 255.255.255.0 U 0 0  tapd568ba1a-74
>
> 172.16.101.0 * 255.255.255.0 U 0 0  tape17162c5-00



既然不需要 namespace 也可以路由，为什么还要加一层 namespace 增加复杂性呢？



其根本原因是：**为了支持网络重叠**。



云环境下，租户可以按照自己的规划创建网络，不同租户的网络是可能重叠的。将路由功能放到 namespace 中，就能隔离不同租户的网络，从而支持网络重叠。



下面通过例子进一步解释。

> Tenant A  vlan100 subnet A-1: 10.10.1.0/24   {"start": "10.10.1.1", "end": "10.10.1.254"}
>
> Tenant A  vlan101 subnet A-2: 10.10.2.0/24   {"start": "10.10.2.1", "end": "10.10.2.254"}
>
> 
>
> Tenant B  vlan102 subnet B-1: 10.10.1.0/24   {"start": "10.10.1.1", "end": "10.10.1.254"}
>
> Tenant B  vlan103 subnet B-2: 10.10.2.0/24   {"start": "10.10.2.1", "end": "10.10.2.254"}



A，B 两个租户定义了完全相同的两个 subnet，网络完全重叠。

其特征是网关 IP 配置在 TAP interface 上。因为没有 namespace 隔离，router_100_101 和 router_102_103 的路由条目都只能记录到控制节点操作系统（root namespace）的路由表中，内容如下：

> Destination Gateway Genmask Flags Metric Use Iface
>
>  10.10.1.0  * 255.255.255.0  U   0    0    tap1
>
>  10.10.2.0  * 255.255.255.0  U   0    0    tap2
>
>  10.10.1.0  * 255.255.255.0  U   0    0    tap3
>
>  10.10.2.0  * 255.255.255.0  U   0    0    tap4



这样的路由表是无法工作的。



按照路由表优先匹配原则，Tenant B 的数据包总是错误地被 Tenant A 的 router 路由。例如 vlan102 上有数据包要发到 vlan103。选择路由时，会匹配路由表的第二个条目，结果数据被错误地发到了 vlan101。



**使用 namespace 的场景**

其特征是网关 IP 配置在 namespace 中的 veth interface 上。每个 namespace 拥有自己的路由表。





router_100_101 的路由表内容如下：

> Destination Gateway Genmask Flags Metric Use Iface
>
> 10.10.1.0 * 255.255.255.0  U   0    0   qr-1
>
> 10.10.2.0 * 255.255.255.0  U   0    0   qr-2



router_102_103 的路由表内容如下：

> Destination Gateway Genmask Flags Metric Use Iface
>
> 10.10.1.0 * 255.255.255.0  U   0    0   qr-3
>
> 10.10.2.0 * 255.255.255.0  U   0    0   qr-4



这样的路由表是可以工作的。



例如 vlan102 上有数据包要发到 vlan103。选择路由时，会查看 router_102_103 的路由表, 匹配第二个条目，数据通过 qr-4 

被正确地发送到 vlan103。



同样当 vlan100 上有数据包要发到 vlan101时，会匹配 router_100_101 路由表的第二个条目，数据通过 qr-2 被正确地发送到 vlan101。



可见，namespace 使得每个 router 有自己的路由表，而且不会与其他 router 冲突，所以能很好地支持网络重叠。



建立外部网络，伴随生成一个网桥br，外部网络网卡接到这个网桥上。

router 的每个 interface 在 namespace 中都有对应的 veth。如果 veth 用于连接租户网络，命名格式为 qr-xxx，比如 qr-d568ba1a-74 和 qr-e17162c5-00。如果 veth 用于连接外部网络，命名格式为 qg-xxx，比如 qg-b8b32a88-03。

![](https://pic.downk.cc/item/5eba4774c2a9a83be57d8526.png)