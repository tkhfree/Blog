---
title: ONOS+mininet实验
thumbnail: 'https://sc03.alicdn.com/kf/H1e0cc92efb194df982b6716e5896f6b1z.jpg'
tags:
  - ONOS
comments: true
toc: true
mathjax: true
abbrlink: 8f8ae963
date: 2020-06-18 15:14:52
urlname:
categories:
---

根据onos官网的例子使用mininet构建简单的网络拓扑图进行试验

![](https://sc03.alicdn.com/kf/H1e0cc92efb194df982b6716e5896f6b1z.jpg)

虽然使用mininet构建好了网络拓扑，但是h1 ping h2并不能成功，因为onos没有向交换机s1下发流表，也就是no flows installed on the data-plane。没有办法转发数据帧。

onos里有一个Reactive Forwarding应用程序，该app可以按需安装转发flows，需要激活

`onos>apps -a -s` 

显示处于活动状态的app

`onos> app activate org.onosproject.fwd`

激活reactive forwarding/ 或者在gui启动

## ONOS CLI命令

#### 设备命令

`onos>devices`

返回系统中已知的设备信息

#### 链接命令

`onos>links`

列出onos里的链接

#### 主机命令

`onos>hosts`

列出onos里的主机

#### 流命令

`onos>flows`

可以观察到在系统中注册的流，可以是以下几种状态：

- PENDING_ADD 流已提交并转发到交换机
- ADDed 流已经加到交换机
- Pending_REMOVE删除流的请求已经提交并转发到交换机
- REMOVED移除规则

#### 路径命令

`onos>path <table>`

table都是交换机设备id，可以自动补全，路径命令可以直观的看到任何两个节点之间的最短路径

#### 意向命令

intent处于以下几个状态

- submitted提交意图并很快执行
- compiling编译，过渡状态
- installing 安装意图
- installed安装完成
- recompiling重新编译
- withdrawing正在撤销
- withdrawn已经撤销
- failed失败

### Intent实验

使用Intent代替flow对网络进行编程，Intent会跟踪网络的状态并重新配置自身已满足你的目的。比如说链接断开，意图框架会将你的意图（也就是流）重新路由到替代路径上，如果没有其他路径，则意图会显示failed状态。

先关闭fwd

`onos>app deactivate fwd`

加入我们的intent

`onos>add-host-intent <tab> <tab>`

tab是主机设备id

可以用intents命令查看安装状态

再去ping这两个主机就可以看到成功了

如果断开一些交换机路径

`mininet>link s2 s11 down`

则onos intent会自动寻找可以建立链接的路径，如果没有则保持在failed状态

删除intent使用

`onos>remove-intent <tab-app> <tab-id>`

Tab-app是应用id， tab-id是intent的id，自动补全

[参考1](http://blog.chinaunix.net/uid-31410005-id-5825102.html)

[参考2](https://www.kancloud.cn/kubee/onosguide/203183)

### 下发流表实验

#### mininet搭建拓扑网络

```shell
sudo mn --topo single,3 --mac --switch ovsk --controller remote，--ip=172.17.0.5 --port=6653
```

创建了三个实例主机和交换机以及一个控制器

可以打开每个主机的终端

```shell
mininet>xterm h1 h2
```

这时候因为流表为空，所以目前ping不通，可以使用dpctl在VMssh终端中手动安装一些流表。

```shell
dpctl add-flow 
```

