---
title: Neutron的基础知识-什么是Open vSwitch
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: a43d924f
date: 2020-05-26 21:09:10
urlname:
categories:
thumbnail:
---

### 如何理解网桥

Bridge（桥）是Linux上用来做TCP/IP二层协议交换的设备，与现实世界中的交换机功能相似。Bridge设备实例可以和Linux上其他网络设备实例连接，既attach一个从设备，类似于在现实世界中的交换机和一个用户终端之间连接一根网线。当有数据到达时，Bridge会根据报文中的MAC信息进行广播、转发、丢弃处理。

### Open vSwitch

Open vSwitch的目标，是做一个具有产品级质量的多层虚拟交换机。通过可编程扩展，可以实现大规模网络的自动化（配置、管理、维护）。它支持现有标准管理接口和协议（比如netFlow，sFlow，SPAN，RSPAN，CLI，LACP，802.1ag等，熟悉物理网络维护的管理员可以毫不费力地通过Open vSwitch转向虚拟网络管理）。总的来说，它被设计为支持分布在多个物理服务器，例如VMware的vNetwork分布式vSwitch或思科的Nexus1000V。

那么什么是虚拟交换？虚拟交换就是利用虚拟平台，通过软件的方式形成交换机部件。跟传统的物理交换机相比，虚拟交换机同样具备众多优点，一是配置更加灵活。一台普通的服务器可以配置出数十台甚至上百台虚拟交换机，且端口数目可以灵活选择。例如，VMware的ESX一台服务器可以仿真出248台虚拟交换机，且每台交换机预设虚拟端口即可达56个；二是成本更加低廉，通过虚拟交换往往可以获得昂贵的普通交换机才能达到的性能，例如微软的Hyper-V平台，虚拟机与虚拟交换机之间的联机速度轻易可达10Gbps



通俗来讲，Open vSwitch是一个由Nicira Networks主导的开源项目，通过运行在虚拟化平台上的虚拟交换机，为本台物理机上的VM提供二层网络接入， 跟云中的其它物理交换机一样工作在Layer 2层。Open vSwitch充分考虑了在不同虚拟化平台间的移植性，采用平台无关的C语言开发。

### Open vSwitch的作用

你可能会问，我为什么有必要在自己的云架构中使用它呢？它能给我的云带来什么？

OK。需求决定一切，如果你只是自己搞一台Host，在上面虚拟几台VM做实验。或者小型创业公司，通过在五台十台机器上的虚拟化，创建一些VM给公司内部开发测试团队使用。那么对你而言，网络虚拟化的迫切性并不强烈。也许你更多考虑的，是VM的可靠接入：和物理机一样有效获取网络连接，能够RDP访问。Linux Kernel自带的桥接模块就可以很好的解决这一问题。原理上讲，正确配置桥接，并把VM的virtual nic连接在桥接器上就OK啦。很多虚拟化平台的早期解决方案也是如此，自动配置并以向用户透明的方式提供虚拟机接入。如果你是OpenStack的fans，那Nova就更好地帮你完成了一系列网络接入设置。Open vSwitch在WHY-OVS这篇文章中，第一句话就高度赞扬了Linux bridge：

“We love the existing network stack in Linux. It is robust, flexible, and feature rich. Linux already contains an in-kernel L2 switch (the Linux bridge) which can be used by VMs for inter-VM communication. So, it is reasonable to ask why there is a need for a new network switch.”

但是，如果你是大型数据中心的网络管理员，一朵没有网络虚拟化支持的云，将是无尽的噩梦。

在传统数据中心中，网络管理员习惯了每台物理机的网络接入均可见并且可配置。通过在交换机某端口的策略配置，可以很好控制指定物理机的网络接入，访问策略，网络隔离，流量监控，数据包分析，Qos配置，流量优化等。



有了云，网络管理员仍然期望能以per OS/per port的方式管理。如果没有网络虚拟化技术的支持，管理员只能看到被桥接的物理网卡，其上川流不息地跑着n台VM的数据包。仅凭物理交换机支持，管理员无法区分这些包属于哪个OS哪个用户，只能望云兴叹乎？简单列举常见的几种需求，Open vSwitch现有版本很好地解决了这些需求。

**需求一**：网络隔离。物理网络管理员早已习惯了把不同的用户组放在不同的VLAN中，例如研发部门、销售部门、财务部门，做到二层网络隔离。Open vSwitch通过在host上虚拟出一个软件交换机，等于在物理交换机上级联了一台新的交换机，所有VM通过级联交换机接入，让管理员能够像配置物理交换机一样把同一台host上的众多VM分配到不同VLAN中去；



  **需求二**：QoS配置。在共享同一个物理网卡的众多VM中，我们期望给每台VM配置不同的速度和带宽，以保证核心业务VM的网络性能。通过在Open vSwitch端口上，给各个VM配置QoS，可以实现物理交换机的traffic queuing和traffic shaping功能。



  **需求三**：流量监控，Netflow，sFlow。物理交换机通过xxFlow技术对数据包采样，记录关键域，发往Analyzer处理。进而实现包括网络监控、应用软件监控、用户监控、网络规划、安全分析、会计和结算、以及网络流量数据库分析和挖掘在内的各项操作。例如，NetFlow流量统计可以采集的数据非常丰富，包括：数据流时戳、源IP地址和目的IP地址、 源端口号和目的端口号、输入接口号和输出接口号、下一跳IP地址、信息流中的总字节数、信息流中的数据包数量、信息流中的第一个和最后一个数据包时戳、源AS和目的AS，及前置掩码序号等。xxFlow因其方便、快捷、动态、高效的特点，为越来越多的网管人员所接受，成为互联网安全管理的重要手段，特别是在较大网络的管理中，更能体现出其独特优势。没错，有了Open vSwitch，作为网管的你，可以把xxFlow的强大淋漓尽致地应用在VM上！

 **需求四**：数据包分析，Packet Mirror。物理交换机的一大卖点，当对某一端口的数据包感兴趣时（for trouble shooting , etc），可以配置各种span（SPAN, RSPAN, ERSPAN），把该端口的数据包复制转发到指定端口，通过抓包工具进行分析。Open vSwitch官网列出了对SPAN, RSPAN, and GRE-tunneled mirrors的支持。

只是在Open vSwitch上实现物理交换机的现有功能？那绝对不是Nicira的风格。



云中的网络，绝不仅仅需要传统物理交换机已有的功能。云对网络的需求，推动了Software Defined Network越来越火。而在各种SDN解决方案中，OpenFlow无疑是最引人瞩目的。Flow Table + Controller的架构，为新服务新协议提供了绝佳的开放性平台。Nicira把对Openflow的支持引入了Open vSwitch。引入以下模块：



·      ovs-openflowd --- OpenFlow交换机；

·      ovs-controller --- OpenFlow控制器；

·      ovs-ofctl --- Open Flow 的命令行配置接口；

·      ovs-pki --- 创建和管理公钥框架；

·      tcpdump的补丁 --- 解析OpenFlow的消息；