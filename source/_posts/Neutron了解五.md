---
title: Neutron了解五
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: a84bf2d1
date: 2020-05-13 11:48:39
urlname:
categories:
thumbnail:
---



**Openvswitch(OVS)在Neutron里实现**

**控制节点**

**br-ex
**连接外部（external）网络的网桥。



**br-int
**集成（integration）网桥，所有 instance 的虚拟网卡和其他虚拟网络设备都将连接到该网桥。



**br-tun
**隧道（tunnel）网桥，基于隧道技术的 VxLAN 和 GRE 网络将使用该网桥进行通信。



这些网桥都是 Neutron 自动为我们创建的，但是通过 brctl show 命令却看不到它们。这是因为我们使用的是 Open vSwitch 而非 Linux Bridge，需要用 Open vSwitch 的命令 ovs-vsctl show 查看。



**计算节点**

计算节点上也有 br-int 和 br-tun，但没有 br-ext。这是合理的，因为发送到外网的流量是通过网络节点上的虚拟路由器转发出去的，所以 br-ext 只会放在网络节点（devstack-controller）上。

---



在 Open vSwitch 环境中，一个数据包从 instance 发送到物理网卡大致会经过下面几个类型的设备：



**tap interface**

命名为 tapXXXX。



**linux bridge**

命名为 qbrXXXX。



**veth pair**

命名为 qvbXXXX, qvoXXXX。



**OVS integration bridge**

命名为 br-int。



**OVS patch ports**

命名为 int-br-ethX 和 phy-br-ethX（X 为 interface 的序号）。



**OVS provider bridge**

命名为 br-ethX（X 为 interface 的序号）。



**物理 interface**

命名为 ethX（X 为 interface 的序号）。



**OVS tunnel bridge**

命名为 br-tun。



OVS provider bridge 会在 flat 和 vlan 网络中使用；OVS tunnel bridge 则会在 vxlan 和 gre 网络中使用。后面会通过实例详细讨论这些设备。



Open vSwitch 支持 local, flat, vlan, vxlan 和 gre 所有五种 network type。vxlan 和 gre 非常类似，