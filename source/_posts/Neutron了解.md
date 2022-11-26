---
title: Neutron了解一
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: 80a7442e
date: 2020-05-09 16:56:40
urlname:
categories:
thumbnail:
---

## neutron创建网络过程

1. 首先管理员拿到一组可以在互联网上寻址的 IP 地址，并且创建一个外部网络和子网。
2. 租户创建一个网络和子网。
3. 租户创建一个路由器并且连接租户子网和外部网络。
4. 租户创建虚拟机。

## OpenStack 网络类型

- 管理网络：用于 OpenStack 各组件之间的内部通信。
- 数据网络：用于云部署中虚拟数据之间的通信。
- 外部网络：公共网络，外部或 internet 可以访问的网络。
- API 网络：暴露所有的 OpenStack APIs,包括 OpenStack 网络 API 给租户们。



## Neutron网络基本概念

Neutron管理的网络资源包括三个：network、subnet、port

- **network**

  network是一个隔离的二级广播域，只在链路层广播。neutron支持的类型local，flat，vlan，vxlan和gre。

  - **local**

  ​       local是只与本机节点上同一网络的实例进行通讯，一般用于单机测试。

  - **flat**

    flat是无valn tagging的网络，可以与不同节点同一网络通讯。

    <font size=2>vlan taging是对于不同vlan n的标识，不同的vlan不能相互通讯，如果有交换机通过Trunk口进行通讯，则不同vlan可以通过tag可以识别哪一个vlan。这里flat就是一个没有Truck口的vlan。</font>

  - **vlan**

    vlan 网络是具有 802.1q tagging 的网络。

    vlan 是一个二层的广播域，同一 vlan 中的 instance 可以通信，不同 vlan 只能通过 router 通信。

    vlan 网络可跨节点，是应用最广泛的网络类型。

  - **vxlan**

    vxlan 是基于隧道技术的 overlay 网络。

    vxlan 网络通过唯一的 segmentation ID（也叫 VNI）与其他 vxlan 网络区分。

    vxlan 中数据包会通过 VNI 封装成 UDP 包进行传输。

    因为二层的包通过封装在三层传输，能够克服 vlan 和物理网络基础设施的限制。

  - **gre**

    gre 是与 vxlan 类似的一种 overlay 网络。

    主要区别在于使用 IP 包而非 UDP 进行封装。

  不同 network 之间在二层上是隔离的。

​		以 vlan 网络为例：

​		network A 和 network B 会分配不同的 VLAN ID，这样就保证了 network A 中的广播包不会跑到 network B 中。

​		当然，这里的隔离是指二层上的隔离，借助路由器不同 network 是可能在三层上通信的。

​		network 必须属于某个 Project（ Tenant 租户），Project 中可以创建多个 network。

​		Project 与 network 之间是 1对多关系。

- **subnet**

  subnet 是一个 IPv4 或者 IPv6 地址段。

  instance 的 IP 从 subnet 中分配。

  每个 subnet 需要定义 IP 地址的范围和掩码。

  network 与 subnet 是 1对多 关系。

  一个 subnet 只能属于某个 network；一个 network 可以有多个 subnet，这些 subnet 可以是不同的 IP 段，但不能重叠。

   

  下面的配置是有效的：

  network A  subnet A-a: 10.10.1.0/24 {"start": "10.10.1.1", "end": "10.10.1.50"}

  ​          subnet A-b: 10.10.2.0/24 {"start": "10.10.2.1", "end": "10.10.2.50"}

   

  但下面的配置则无效，因为 subnet 有重叠

  network A  subnet A-a: 10.10.1.0/24 {"start": "10.10.1.1", "end": "10.10.1.50"}

  ​       subnet A-b: 10.10.1.0/24 {"start": "10.10.1.51", "end": "10.10.1.100"}

   

  这里不是判断 IP 是否有重叠，而是 subnet 的 CIDR 重叠（都是 10.10.1.0/24）。

  但是，如果 subnet 在不同的 network 中，CIDR 和 IP 都是可以重叠的，比如

  network A  subnet A-a: 10.10.1.0/24 {"start": "10.10.1.1", "end": "10.10.1.50"}

  networkB  subnet B-a: 10.10.1.0/24 {"start": "10.10.1.1", "end": "10.10.1.50"}

  如果上面的IP地址是可以重叠的，那么就可能存在具有相同 IP 的两个 instance，这样并不会冲突。

   

  具体原因：

  因为 Neutron 的 router 是通过 Linux network namespace 实现的。

  network namespace 是一种网络的隔离机制。

  通过它，每个 router 有自己独立的路由表。

   

  上面的配置有两种结果：

  如果两个 subnet 是通过同一个 router 路由，根据 router 的配置，只有指定的一个 subnet 可被路由。

  如果上面的两个 subnet 是通过不同 router 路由，因为 router 的路由表是独立的，所以两个 subnet 都可以被路由。

- **port**

  port 可以看做虚拟交换机上的一个端口。

  port 上定义了 MAC 地址和 IP 地址，当 instance 的虚拟网卡 VIF（Virtual Interface） 绑定到 port 时，port 会将 MAC 和 IP 分配给 VIF。

  subnet 与 port 是 1对多 的关系：

  一个 port 必须属于某个 subnet；

  一个 subnet 可以有多个 port。

- **小节**

  下面总结了 Project，Network，Subnet，Port 和 VIF 之间关系。

  Project 1 : m Network

  Network 1 : m Subnet

  Subnet 1 : m Port

  Port 1 : 1 VIF

  VIF m : 1 Instance



-----



**Neutron** **架构**

与 OpenStack 的其他服务的设计思路一样，Neutron 也是采用分布式架构，由多个组件（子服务）共同对外提供网络服务。

 

Neutron 由如下组件构成：

**Neutron Server**

　　对外提供 OpenStack 网络 API，接收请求，并调用 Plugin 处理请求。

**Plugin**

　　处理 Neutron Server 发来的请求，维护 OpenStack 逻辑网络状态， 并调用 Agent 处理请求。

**Agent**

　　处理 Plugin 的请求，负责在 network provider 上真正实现各种网络功能。

**network provider**

　　提供网络服务的虚拟或物理网络设备，例如 Linux Bridge，Open vSwitch 或者其他支持 Neutron 的物理交换机。

**Queue**

　　Neutron Server，Plugin 和 Agent 之间通过 Messaging Queue 通信和调用。

**Database**

　　存放 OpenStack 的网络状态信息，包括 Network, Subnet, Port, Router 等。

 

 

Neutron 架构非常灵活，层次较多，目的是：

为了支持各种现有或者将来会出现的优秀网络技术。

支持分布式部署，获得足够的扩展性。

 

以创建一个 VLAN100 的 network 为例，假设 network provider 是 linux bridge， 流程如下：

- Neutron Server 接收到创建 network 的请求，通过 Message Queue（RabbitMQ）通知已注册的 Linux Bridge Plugin。
- Plugin 将要创建的 network 的信息（例如名称、VLAN ID等）保存到数据库中，并通过 Message Queue 通知运行在各节点上的 Agent。
- Agent 收到消息后会在节点上的物理网卡（比如 eth2）上创建 VLAN 设备（比如 eth2.100），并创建 bridge （比如 brqXXX） 桥接 VLAN 设备。

---

**ML2**

ML2 对二层网络进行抽象和建模，引入了 type driver 和 mechansim driver。

**Type Driver**

Neutron 支持的每一种网络类型都有一个对应的 ML2 type driver。

type driver 负责维护网络类型的状态，执行验证，创建网络等。

ML2 支持的网络类型包括 local, flat, vlan, vxlan 和 gre。

 

**Mechansim Driver**

Neutron 支持的每一种网络机制都有一个对应的 ML2 mechansim driver。

mechanism driver 负责获取由 type driver 维护的网络状态，并确保在相应的网络设备（物理或虚拟）上正确实现这些状态。

 

type 和 mechanisim 都太抽象，现在我们举一个具体的例子：

<font color=blue>type driver 为 vlan，mechansim driver 为 linux bridge</font>

<font color=blue>我们要完成的操作是创建 network vlan100，那么：vlan type driver 会确保将 vlan100 的信息保存到 Neutron 数据库中，包括 network 的名称，vlan ID 等。</font>

<font color=blue>linux bridge mechanism driver 会确保各节点上的 linux brige agent 在物理网卡上创建 ID 为 100 的 vlan 设备 和 brige 设备，并将两者进行桥接。</font>

 

mechanism driver 有三种类型：

**Agent-based**

包括 linux bridge, open vswitch 等。

**Controller-based**

包括 OpenDaylight, VMWare NSX 等。

**基于物理交换机**

包括 Cisco Nexus, Arista, Mellanox 等。

比如前面那个例子如果换成 Cisco 的 mechanism driver，则会在 Cisco 物理交换机的指定 trunk 端口上添加 vlan100。

 

本章的 mechanism driver 将涉及 linux bridge, open vswitch 和 L2 population。

linux bridge 和 open vswitch 的 ML2 mechanism driver 作用是配置各节点上的虚拟交换机。

linux bridge driver 支持的 type 包括 local, flat, vlan, vxlan。

open vswitch driver 支持的 type 包括 local, flat, vlan, vxlan, gre。

 

L2 population driver 作用是优化和限制 overlay 网络中的广播流量。 vxlan 和 gre 都属于 overlay 网络。

 

ML2 core plugin 已经成为 OpenStack Neutron 的首选 plugin 。

----

1、Neutron 通过 plugin 和 agent 提供的网络服务。

2、plugin 位于 Neutron server，包括 core plugin 和 service plugin。

3、agent 位于各个节点，负责实现网络服务。

4、core plugin 提供 L2 功能，ML2 是推荐的 plugin。

5、使用最广泛的 L2 agent 是 linux bridage 和 open vswitch。

6、service plugin 和 agent 提供扩展功能，包括 dhcp, routing, load balance, firewall, vpn 等。

![](https://pic.downk.cc/item/5eb90decc2a9a83be512046f.png)