---
title: SDN定义概念
tags:
  - OpenStack
  - SDN
comments: true
toc: true
mathjax: true
abbrlink: 4e452ca7
date: 2020-05-08 11:33:37
urlname:
categories:
thumbnail:
---



传统网络就像真人在踢足球，每个人自己思考下一步把球传到哪儿，然后自己用头或者脚传出去。SDN就像你在打实况足球，球员怎么传球都是集中由你来控制。这就叫控制面和转发面的分离，并且控制面集中起来。集中后的控制面叫做控制器，类似游戏手柄。控制器之上就是软件，来 操作控制器具体怎么处理。就像你聪明的大脑来控制游戏手柄。open flow就是控制器与网络设备沟通时的语言。



作者：陈博
链接：https://www.zhihu.com/question/20279620/answer/23434541
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

----



**SDN**是一种思想，不是任何协议或者技术，是一种把专用电信硬件采用x86通用硬件替代的思想，专用电信硬件所支持的各种协议，通过sdn思想，软件编程化，即软件定义网络（soft defined network）。

只要网络硬件可以集中软件式管理、可编程、转发和控制分开，则认为这个网络是一个sdn网络。



作者：int32bit
链接：https://www.zhihu.com/question/37126320/answer/382480862
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



首先从大的角度谈区别，OpenStack是一个IaaS开源实现方案之一，提供计算、存储、网络等基础服务以及诸如数据库服务、大数据服务、容器编排服务等高级服务，而SDN主要聚集在网络这一块。因此从技术生态覆盖面角度看，OpenStack的生态方方面面十分庞大，网络只是其中的一个点，由组件Neutron负责实现。而SDN的生态只有网络这一点。

同样是网络，**OpenStack Neutron注重的是为虚拟机、容器、物理服务器等提供网络服务**，提供的是面向用户的北向RESTFul API，而**SDN注重的是解决网络设备的配置以及转发控制问题**，更侧重于底层网络设备的配置管理以及控制，既要提供面向用户的北向API，还要提供南向API，对接各种硬件设备的协议。

更细化的角度看联系，SDN的思想是控制转发分离，这是一种网络设计理念和变革。具体什么是控制转发分离，google有很专业的描述。我的简单理解就是把以前需要手动ssh或者通过console一台一台配置交换机、路由器、防火墙、负载均衡等硬件的活，现在通过一个中心控制器统一纳管起来，对用户屏蔽了底层硬件区别，网络管理员只需要调用标准API或者Web界面操作控制器就能自动下发配置策略到指定的设备中。当然除了配置问题，有些功能也从网络设备中剥离出来，放到控制器实现，不再揉到一块，这些功能包括链路发现、mac地址学习、路由计算等。硬件只负责转发或者路由即可。控制器如何与硬件打交道呢？当然是通过协议了，如openflow、netconf等。这里我们只关注SDN中的控制器，显然这是一个调度与控制中心，向上对用户负责，为用户提供API，向下对接各种网络设备，下发策略。

我们再来看OpenStack Neutron，我们知道，OpenStack Neutron由neutron-server和一堆agents/drivers组成。这个neutron-server的功能和SDN中的控制器非常类似，向上对用户负责，为用户提供可编程API，向下对接各种agents，只是对接的协议不同，neutron-server与agent的交互的南向协议是RPC。所以我们可以简单理解neutron-server其实就是SDN控制器实现之一（不一定准确），负责向各种agents下发策略，agents再向各种网络设备转发策略。问题是除了虚拟网络设备（如OVS），还有各种各样的硬件网络设备，这些网络设备的功能、配置标准以及协议等均存在很大的差别，如果OpenStack Neutron要全部纳管起来，需要写一堆的agents和drivers，而且是重复造轮子，功能上也没有解耦，Neutron会非常重。最好的方式是不要直接去对接硬件，直接对接现成的SDN控制器即可，因为SDN控制器提供的北向API通常是标准的，而又屏蔽了底层硬件设备的差异化，不再需要实现各种各种的agents和drivers了，只需要实现对接SDN控制器的agent或者driver即可，比如对接开源的SDN控制器ODL([openstack/networking-odl](https://link.zhihu.com/?target=https%3A//github.com/openstack/networking-odl))以及华为AC控制器等等。

简而言之，SDN是一种思想，核心理念是控制与转发分离，控制由控制器实现，转发由网络设备实现，其中ODL是控制器的开源实现之一。OpenStack Neutron是为OpenStack提供网络服务的组件，neutron-server服务可以当作是SDN控制器的实现之一，Neutorn可以串接其它的SDN控制器，比如ODL。

另外，如果想从事SDN或者OpenStack开发，并不是一定要两者都懂，看你负责什么工作了。OpenStack的大多数开发都不需要直接接触SDN，比如你负责存储服务Cinder或者计算服务Nova，基本就不会直接与SDN打交道。同样的，从事SDN也不一定非要和OpenStack扯上关系，二者可以说是完全解耦独立的。当然，如果你负责OpenStack Neutron开发，并且需要对接SDN，则对二者的熟悉是必须的了。