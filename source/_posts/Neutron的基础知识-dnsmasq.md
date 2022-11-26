---
title: Neutron的基础知识-dnsmasq
tags:
  - Neutron
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: a49f2fbc
date: 2020-05-26 14:10:59
urlname:
categories:
thumbnail:
---

openstack概念中，我们会看到Dnsmasq，那么它的作用是什么？简单来讲，是为了提高dns性能的。
混杂模式又分为集线器模式，及交换机模式，这里不在细分，只是对这个模式有一个简单的感性认知即可.

### dnsmasq

提供 DNS 缓存和 DHCP 服务功能。作为域名解析服务器(DNS)，dnsmasq可以通过缓存 DNS 请求来提高对访问过的网址的连接速度。作为DHCP 服务器，dnsmasq 可以为局域网电脑提供内网ip地址和路由。DNS和DHCP两个功能可以同时或分别单独实现。dnsmasq轻量且易配置，适用于个人用户或少于50台主机的网络。

### 混杂模式（Promiscuous Mode）

是指一台机器能够接收所有经过它的数据流，而不论其目的地址是否是他。是相对于通常模式（又称“非混杂模式”）而言的。这被网络管理员使用来诊断网络问题，但是也被无认证的想偷听网络通信（其可能包括密码和其它敏感的信息）的人利用。

