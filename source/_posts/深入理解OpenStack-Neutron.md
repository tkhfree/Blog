---
title: 深入理解OpenStack Neutron
tags:
  - OpenStack
  - Neutron
comments: true
toc: true
mathjax: true
abbrlink: cf7eca02
date: 2020-06-01 14:44:59
urlname:
categories:
thumbnail:
---

# Linux虚拟网络基础

对于《深入理解OpenStack Neutron》这本书的今日理解：

Linux虚拟网络里面的设备跟我们通俗意义理解的设备并不一样，物理设备可能就是一个具有某些功能的物理实体。Linux虚拟网络里创建的设备就是一个结构体、驱动程序等。虚拟网络设备具有的一些功能可以理解为具有网桥、交换机、网卡功能的虚拟化，但是又不能直接把他们认为是网卡等设备，他们本质上是一个结构体，是一段函数定义，比如tap/tun，定义完全一样，只是flag有区别，定义他们在二层还是在三层工作。

**tap、tun、veth pair在Linux中都被称为设备，在日常概念里，通常被称为接口。**

**反而是Bridge、Router这些在Linux中没有被称为设备的网络功能，在日常概念里被称为设备。**

# Neutron架构分析

## Neutron的web server框架

采用pecan框架，遵循WSGI规范，参考《WSGI网关协议理解》

## Neutron的进程之间消息通信机制

AMQP标准，product和consumer的PRC调用机制，参考《thrift了解》

## Neutron的并发机制

Neutron使用Python编程，使用CPython解释器，一个进程内只能运行一个线程。

Neutron不属于”计算密集型应用“，属于”I/O阻塞型应用“，不使用多进程而使用协程反而有一定的优势。

#### 协程概述

协程不是进程，也不是线程，一个进程可以有多个线程，一个线程可以有多个协程，协程是用户态，不牵涉到内核态，可以认为它是一个可以挂起的函数，既然是函数就说明协程是串行的，也就是说不管有几核的cpu，都是使用单核运行多个协程

I/O阻塞型是指相比于cpu的处理速度，当程序运行在I/O相关程序时，cpu时空闲的，因此可以把函数挂起，处理其他的协程

当处理的I/O程序比较多，属于”I/O密集型“，使用协程无法应付，这时候采用”多进程+协程“

Neutron采用协程主要是使用yield和eventlet协程库

- yield：可以参考《Python总结一》

- eventlet：是对使用yield这种主动的协程调度进行封装，对用户（应用程序）不可见，eventlet是对greenlet进行封装。

  - greenlet

    ```python
    def func1():
        print ("t1.1")
        gr2.switch()
        print ("t1.2")
    def func2():
        print ("t2.1")
        gr1.switch()
        print ("t2.2")
    gr1 = greenlet(func1) #创建协程对象，func1在协程gr1里运行
    gr2 = greenlet(func2)
    gr1.switch()
    
    # 协程对象被创建后并不会主动运行，需要成员函数switch（）去调度。
    # gr1.switch()被调度，在协程对象1里执行函数func1，打印t1.1,执行gr2.switch()，func1被挂起，执行协程对象2里的func2，打印t2.1，在执行gr1.switch()，func1被调度，继续被挂起的地方执行，打印t1.2，执行完毕执行被挂起的协程对象gr2，打印t2.2，因此最终输出结果：
    # t1.1 t2.1 t1.2 t2.2
    ```

  - eventlet：是对greenlet的封装，greenlet需要编程者去主动调度协程，eventlet不仅封装协程对象，还封装协程的切换（调度）。它还是一个网络处理方面的协程库，因为它是非阻塞型I/O协程，当I/O不发生阻塞时，他不会切到其他协程，只有网络I/O发生阻塞，eventlet就会调度另一个协程，直到这个协程发生阻塞或者运行结束。

    ```python
    import eventlet
    import time 
    
    def test(s):
        print (s+"begin")
        time.sleep(1)
        print (s+"end")
        
    pool = eventlet.GreenPool(3) #起三个空的协程池，称为greenthread
    for i in range(3):
        pool.spawn(test(str(i))) #pool.spawn创建一个具体的协程，执行函数是test
        
    ---------
    0 begin
    0 end
    1 begin
    1 end
    2 begin
    2 end
    ```

    

    

    

