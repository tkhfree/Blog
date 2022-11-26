---
title: onos学习一
toc: true
comments: true
thumbnail: 'https://pic.downk.cc/item/5ee896022cb53f50feba89df.png'
tags:
  - ONOS
mathjax: true
abbrlink: ee728f2f
date: 2020-06-16 10:46:01
urlname:
categories:
---

一周没有总结了，上周划水好爽

今天把服务器部署好，这周开始学习一下onos以及下发流表的一些东西。

ML2(model layer 2)是一种允许openstack网络同时利用多种二层网络技术的框架，ML2包括两种驱动类型，类型驱动(Type driver)和机制驱动(Mechanism Driver)分别实现可扩展的网络类型和访问这些网络类型的网络机制集合。类型驱动实现网络类型(GRE、vxlan、vlan)，机制驱动实现网络机制集合(Linux bridge、open vswich、ODL、ONOS)。

[onos的官网](https://wiki.onosproject.org/)

学习onos之前需要先**了解openflow以及用mininet进行演练**，如果做onos开发还需要了解flowvisor和apache karaf。

## openflow

ONF基金会提出的基于OpenFlow协议的三层网络架构

|     **应用平面**     |          |
| :------------------: | :------: |
| **控制平面**（SDN）  | 管理平面 |
| **数据平面（转发）** |          |

在OpenFlow协议中，数据平面被定义为由多级流表（Flow Table）驱动的转发模型

流表1>>>流表2>>>流表3

### 可编程

北向接口可编程 rest api、java api

南向接口可编程openflow

![](https://pic.downk.cc/item/5ee896022cb53f50feba89df.png)

数据平面可编程 DPDk、p4

![](https://pic.downk.cc/item/5ee896c22cb53f50febb785a.png)

### SDN数据层面

执行网络控制逻辑：

1. 解析数据包头
2. 转发数据包到某些端口

openflow交换机模型：

将传统的网络数据平面的各种查找表（mac地址表、路由表）转而用抽象的流表结构表示，而且支持管道流通，将数据转发处理抽象成匹配-动作（match-action）模型

### OpenFlow交换机架构

#### 端口

物理端口

逻辑端口

保留端口

安全通道（sw）：负责控制器与交换机之间的交互，通过安全通道与远端控制器进行链接

流表（hw）：指示交换机如何进行流的处理，每个动作关联一个流表项

OpenFlow协议：南向接口

#### 流表

流是有某些共同特征的数据集合

一张流表包含一系列的流表项

流表项

- 包头域（匹配域）：涵盖了链路层、网络层、传输层大部分表示
- 计数器：统计数据流量相关嘻嘻
- 动作表（指令集）：匹配的数据包应该进行的动作

![](https://pic.downk.cc/item/5ee8ba742cb53f50fedeaa5e.png)

### Open vSwitch

open vswitch是一个软SDN交换机。借助第三方控制器（onos、ODL）或管理工具实现复杂的转发策略。不连接外部控制器，他自身也可以依靠mac地址学习实现二层数据包转发功能，类似于Linux bridge。

Open vSwitch的一些概念：

- Bridge代表以太网交换机（switch）
- 交换机包含端口（port）：**Normal物理网卡、Internal虚拟网卡、Patch连接不同的bridge、Tunnel隧道类型端口**
- 一个端口可以有多个接口（Interfaces）
- 数据包通过流（flow）转发的

![](https://pic.downk.cc/item/5ee8bfaf2cb53f50fee3e699.png)

## 南向接口协议

控制器与交换机之间的通信协议

- 向上提供交换机的状态信息，向下下发控制策略、指导转发行为
- 实现网络的配置与管理
- 实现路径计算

[***sdn南向接口协议除了openflow，为什么需要netconf和ovsdb？***](https://www.zhihu.com/question/267926088/answer/382137375)

***3个协议需要都同时存在吗？各协议负责哪些工作？***

三个协议同属于南向协议，openflow区别与后两个，openflow主要管理流表，交换机状态，但是不支持配置。后两个同属于配置交换机的协议，两个是竞争关系，同样的还有ofconfig协议，主要区别在于ovsdb只支持虚拟交换机，netconf则可以应用与物理机。

至于管理交换机和配置交换机的区别，举个具体的例子，如果需要对某个交换机利用queue进行限速处理，那么你需要先在交换机上配置queue，这时候你就需要用到ovsdb或者ofconfig等。当你的交换机上已经有了queue，这时候你需要将对应的流进行入队操作，这时候你需要的就是openflow协议对流进行管理。

至于是否需要同时存在，个人以为openflow是必不可少的，毕竟是核心协议。至于netconf和ovsdb看具体的业务需要，就算是没有，无非是无法从controller层配置交换机，需要交换机本地配置。

### OpenFlow协议

安全通道（sw）有三种消息类型：Controller-to-Switch（控制器发起）、asynchronous（交换机发起）、symmetric（控制器或者交换机）

OpenFlow的消息格式：OpenFlow Header（8字节）和OpenFlow Message

![](https://pic.downk.cc/item/5ee990bd359fca8463e33bad.jpg)

​												OpenFlow协议下的交换机与控制器交互流程

### OF-CONFIG协议

交换机与控制器进行通讯前，需要对其功能、特性以及资源进行配置。

需要专门的配置协议进行配置，与OpenFlow是伴侣协议

### NETCONFIG协议

也是常用的南向接口之一

## 控制平面

一个或者多个SDN控制器

SDN控制器体系架构：

- [ ] 北向接口层
- [ ] 内置应用层
- [ ] 基础网络层                ——配置管理层
- [ ] 抽象逻辑层
- [ ] 南向接口层

开源SDN控制器：python语言的Ryu、POX；java语言的ODL、ONOS

## 北向接口

例如Floodlight北向API：RESTful API

- 流表OpenFlow

- 防火墙

- ACL

- 多租户网络虚拟化

  