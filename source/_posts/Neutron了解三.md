---
title: Neutron了解三
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: f97bfc8a
date: 2020-05-12 15:19:39
urlname:
categories:
thumbnail:
---

当租户网络连接到 Neutron router，通常将 router 作为默认网关。当 router 接收到 instance 的数据包，并将其转发到外网时:

1. router 会修改包的源地址为自己的外网地址，这样确保数据包转发到外网，并能够从外网返回。



2. router 修改返回的数据包，并转发给真正的 instance。



这个行为被称作 [Source NAT](http://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587316&idx=1&sn=114abfc48e6984309507e523d1548cad&chksm=8d308f6dba47067b5b206331fdbc2391dc508e50697ea26a4012e11763d391dc1f3e1117ff13&scene=21#wechat_redirect)。



如果需要从外网直接访问 instance，则可以利用 floating IP。下面是关于 floating IP 必须知道的事实：



1. floating IP 提供静态 NAT 功能，建立外网 IP 与 instance 租户网络 IP 的一对一映射。



2. floating IP 是配置在 router 提供网关的外网 interface 上的，而非 instance 中。



3. router 会根据通信的方向修改数据包的源或者目的地址。



- floating IP 能够让外网直接访问租户网络中的 instance。这是通过在 router 上应用 iptables 的 NAT 规则实现的。



- floating IP 是配置在 router 的外网 interface 上的，而非 instance，这一点需要特别注意。