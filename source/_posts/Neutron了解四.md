---
title: Neutron了解四
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: b5a99094
date: 2020-05-12 16:05:32
urlname:
categories:
thumbnail:
---

**vxlan** 和 **gre** 两种 overlay network

overlay network 是指建立在其他网络上的网络。overlay network 中的节点可以看作通过虚拟（或逻辑）链路连接起来的。overlay network 在底层可能由若干物理链路组成，但对于节点，不需要关心这些底层实现。



例如 P2P 网络就是 overlay network，隧道也是。vxlan 和 gre 都是基于隧道技术实现的，它们也都是 overlay network。



目前 linux bridge 只支持 vxlan，不支持 gre；open vswitch 两者都支持。vxlan 与 gre 实现非常类似，而且 vxlan 用得较多。



VXLAN 为 Virtual eXtensible Local Area Network。正如名字所描述的，VXLAN 提供与 VLAN 相同的以太网二层服务，但拥有更强的扩展性和灵活性。与 VLAN 相比，VXLAN 有下面几个优势：



1. 支持更多的二层网段。

VLAN 使用 12-bit 标记 VLAN ID，最多支持 4094 个 VLAN，这对大型云部署会成为瓶颈。VXLAN 的 ID （VNI 或者 VNID）则用 24-bit 标记，支持 16777216 个二层网段。



2. 能更好地利用已有的网络路径。

VLAN 使用 Spanning Tree Protocol 避免环路，这会导致有一半的网络路径被 block 掉。VXLAN 的数据包是封装到 UDP 通过三层传输和转发的，可以使用所有的路径。



3. 避免物理交换机 MAC 表耗尽。

由于采用隧道机制，TOR (Top on Rack) 交换机无需在 MAC 表中记录虚拟机的信息。



VXLAN 封装和包格式


VXLAN 是将二层建立在三层上的网络。通过将二层数据封装到 UDP 的方式来扩展数据中心的二层网段数量。


VXLAN 是一种在现有物理网络设施中支持大规模多租户网络环境的解决方案。VXLAN 的传输协议是 IP + UDP。



VXLAN 定义了一个 MAC-in-UDP 的封装格式。在原始的 Layer 2 网络包前加上 VXLAN header，然后放到 UDP 和 IP 包中。通过 MAC-in-UDP 封装，VXLAN 能够在 Layer 3 网络上建立起了一条 Layer 2 的隧道。



VXLAN 包的格式如下：

[![](https://pic.downk.cc/item/5eba61d5c2a9a83be59c02f4.png)](https://pic.downk.cc/item/5eba61d5c2a9a83be59c02f4.png)

​	

如上图所示，VXLAN 引入了 8-byte VXLAN header，其中 VNI 占 24-bit。VXLAN 和原始的 L2 frame 被封装到 UDP 包中。这 24-bit 的 VNI 用于标示不同的二层网段，能够支持 16777216 个 LAN。

VXLAN Tunnel Endpoint

VXLAN 使用 VXLAN tunnel endpoint (VTEP) 设备处理 VXLAN 的封装和解封。每个 VTEP 有一个 IP interface，配置了一个 IP 地址。VTEP 使用该 IP 封装 Layer 2 frame，并通过该 IP interface 传输和接收封装后的 VXLAN 数据包。



下面是 VTEP 的示意图：

[![](https://pic.downk.cc/item/5eba6281c2a9a83be59cae39.png)](https://pic.downk.cc/item/5eba6281c2a9a83be59cae39.png)

**VXLAN 包转发流程**



VXLAN 在 VTEP 间建立隧道，通过 Layer 3 网络传输封装后的 Layer 2 数据。下面例子演示了数据如何在 VXLAN 上传输：

[![](https://pic.downk.cc/item/5eba6478c2a9a83be59e4845.png)](https://pic.downk.cc/item/5eba6478c2a9a83be59e4845.png)

图中 Host-A 和 Host-B 位于 VNI 10 的 VXLAN，通过 VTEP-1 和 VTEP-2 之间建立的 VXLAN 隧道通信。数据传输过程如下：



1. Host-A 向 Host-B 发送数据时，Host-B 的 MAC 和 IP 作为数据包的目标 MAC 和 IP，Host-A 的 MAC 作为数据包的源 MAC 和 IP，然后通过 VTEP-1 将数据发送出去。



2. VTEP-1 从自己维护的映射表中找到 MAC-B 对应的 VTEP-2，然后执行 VXLAN 封装，加上 VXLAN 头，UDP 头，以及外层 IP 和 MAC 头。此时的外层 IP 头，目标地址为 VTEP-2 的 IP，源地址为 VTEP-1 的 IP。同时由于下一跳是 Router-1，所以外层 MAC 头中目标地址为 Router-1 的 MAC。



3. 数据包从 VTEP-1 发送出后，外部网络的路由器会依据外层 IP 头进行路由，最后到达与 VTEP-2 连接的路由器 Router-2。



4. Router-2 将数据包发送给 VTEP-2。VTEP-2 负责解封数据包，依次去掉外层 MAC 头，外层 IP 头，UDP 头 和 VXLAN 头。VTEP-2 依据目标 MAC 地址将数据包发送给 Host-B。


上面的流程我们看到 VTEP 是 VXLAN 的最核心组件，负责数据的封装和解封。

隧道也是建立在 VTEP 之间的，VTEP 负责数据的传送。

**Linux 对 VXLAN 的支持
**



VTEP 可以由专有硬件来实现，也可以使用纯软件实现。目前比较成熟的 VTEP 软件实现包括：

1. 带 VXLAN 内核模块的 Linux

2. Open vSwitch



我们先来看 Linux 如何支持 VXLAN，Open vSwitch 方式将在后面讨论。

[![](https://pic.downk.cc/item/5eba64dcc2a9a83be59e936d.png)](https://pic.downk.cc/item/5eba64dcc2a9a83be59e936d.png)

实现方式：



1. Linux vxlan 创建一个 UDP Socket，默认在 8472 端口监听。



2. Linux vxlan 在 UDP socket 上接收到 vxlan 包后，解包，然后根据其中的 vxlan ID 将它转给某个 vxlan interface，然后再通过它所连接的 linux bridge 转给虚机。



3. Linux vxlan 在收到虚机发来的数据包后，将其封装为多播 UDP 包，从网卡发出。



[<font size=5>实例</font>](https://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587527&idx=1&sn=5ce88aaf61a01a72388619c831f66a85&chksm=8d30805eba4709487a3554f6016498a4e249ade2bb3260c6b7b288cebf13e866080f43f0c3be&scene=21#wechat_redirect)