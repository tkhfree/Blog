---
title: thrift了解与AMQP
tags:
  - Thrift
  - AMQP
comments: true
toc: true
mathjax: true
abbrlink: f9febc9
date: 2020-05-25 22:39:51
urlname:
categories:
thumbnail:
---

# Thrift入门

#### Thrift简介

##### 什么是thrift

简单来说,是Facebook公布的一款开源跨语言的RPC框架.

与此类似的是rabbitMQ，属于AMQP，进行RPC通信

##### 什么是RPC框架?

RPC (Remote Procedure Call Protocal)，远程过程调用协议

RPC, 远程过程调用直观说法就是A通过网络调用B的过程方法。

- 简单的说，RPC就是从一台机器（客户端）上通过参数传递的方式调用另一台机器（服务器）上的一个函数或方法（可以统称为服务）并得到返回的结果。
- RPC 会隐藏底层的通讯细节（不需要直接处理Socket通讯或Http通讯） RPC 是一个请求响应模型。
- 客户端发起请求，服务器返回响应（类似于Http的工作方式） RPC 在使用形式上像调用本地函数（或方法）一样去调用远程的函数（或方法）。

**RPC 框架的目标就是让远程服务调用更加简单、透明**，RPC 框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节。**服务调用者可以像调用本地接口一样调用远程的服务提供者，而不需要关心底层通信细节和调用过程。**

RPC 框架的调用原理图如下所示：

![](https://pic.downk.cc/item/5fb4deffb18d62711359873d.jpg)

早期单机时代，一台电脑上运行多个进程，大家各干各的，老死不相往来。假如A进程需要一个画图的功能，B进程也需要一个画图的功能，程序员就必须为两个进程都写一个画图的功能。这不是整人么？于是就出现了IPC（Inter-process communication，单机中运行的进程之间的相互通信）。OK，现在A既然有了画图的功能，B就调用A进程上的画图功能好了，程序员终于可以偷下懒了。

到了网络时代，大家的电脑都连起来了。以前程序只能调用自己电脑上的进程，能不能调用其他机器上的进程呢？于是就程序员就把IPC扩展到网络上，这就是RPC（远程过程调用）了。现在不仅单机上的进程可以相互通信，多机器中的进程也可以相互通信了。要知道实现RPC很麻烦呀，什么多线程、什么Socket、什么I/O，都是让咱们普通程序员很头疼的事情。于是就有牛人开发出RPC框架（比如，CORBA、RMI、Web Services、RESTful Web Services等等）。OK，现在可以定义RPC框架的概念了。简单点讲，RPC框架就是可以让程序员来调用远程进程上的代码一套工具。有了RPC框架，咱程序员就轻松很多了，终于可以逃离多线程、Socket、I/O的苦海了。

##### thrift的跨语言特型

thrift通过一个中间语言IDL(接口定义语言)来定义RPC的数据类型和接口,这些内容写在以.thrift结尾的文件中,然后通过特殊的编译器来生成不同语言的代码,以满足不同需要的开发者,比如java开发者,就可以生成java代码,c++开发者可以生成c++代码,生成的代码中不但包含目标语言的接口定义,方法,数据类型,还包含有RPC协议层和传输层的实现代码.

##### thrift的协议栈结构

thrift是一种c/s的架构体系。在最上层是用户自行实现的业务逻辑代码。
 　第二层是由thrift编译器自动生成的代码，主要用于结构化数据的解析，发送和接收。TServer主要任务是高效的接受客户端请求，并将请求转发给Processor处理。Processor负责对客户端的请求做出响应，包括RPC请求转发，调用参数解析和用户逻辑调用，返回值写回等处理。
 　从TProtocol以下部分是thirft的传输协议和底层I/O通信。TProtocol是用于数据类型解析的，将结构化数据转化为字节流给TTransport进行传输。TTransport是与底层数据传输密切相关的传输层，负责以字节流方式接收和发送消息体，不关注是什么数据类型。底层IO负责实际的数据传输，包括socket、文件和压缩数据流等。

#### Thrift安装

安装环境：window 7

- 在[官网](https://link.jianshu.com?t=http://archive.apache.org/dist/thrift/0.9.3/)上下载thrift-0.9.3.exe包到一个新建文件夹(博主的文件夹名称为Thrift)中
- 然后将此文件夹放到环境变量Path中。例如博主就是将D:Thrift添加到Path中
- cmd，打开终端，输入`thrift -version`，即可看到相应的版本号，就算是成功安装啦

# AMQP(Advanced message queuing protocol)

RabbitMQ：服务端采用Erlang语言，客户端支持多种语言。进行RPC通讯。

AMQP基本概念：

![](https://pic.downk.cc/item/5f0433f714195aa5947e67cb.png)

## AMQP的消息转发

AMQP server里有两个部件：

- EXChange 接受Product发送的消息，按照一定的规则转发到响应的message Queues中
- message queues再将消息转发到相应的consumers

Product1产生的消息经过AMQP server转发到Consumer_n，起关键作用的路由标识是字符串Routing Key。

每个Product生产消息时都会带上Routing key，而Consumer会告诉AMQP server它希望接受的消息Routing Key时什么。

Message Queue匹配时有三种匹配方式

- direct Exchanges 全值匹配
- Topic Exchanges 通配符匹配，`*`代表一个字符串，`#`代表任意多个子字符串，product生产的字符串是用`.`隔开的子字符串
- Fanout Exchanges 广播匹配，没有Routing key

基于AMQP的以上三种消息转发模型，有三种通信模式：

- RPC(远程过程调用)
- 发布-订阅
- 广播

后两种基于通配符匹配和广播匹配就行了，主要讲一下RPC：

RPC是一种C/S通信模型，当Client发送request时，C是Product，S是Consumer；

当server回复response时，S是Product，C是Consumer。

因此基于AMQP的RPC实现原理如下图：

![](https://pic.downk.cc/item/5f043a0c14195aa59480f992.png)