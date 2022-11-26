---
title: openstack概述
tags:
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: 8afdb7cd
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---

[openstack概述](https://yq.aliyun.com/articles/494442)
[openstack方法论](https://www.cnblogs.com/CloudMan6/p/6391603.html)
[Openstack之keystone源代码分析2--Controller->Manager->Driver](http://www.aboutyun.com/thread-10138-1-1.html)
[Openstack之keystone源代码分析1--WSGI接口流程分析](http://www.aboutyun.com/thread-10137-1-1.html)
[[openstack][G版]keystone源码记录](http://www.aboutyun.com/thread-10136-1-1.html)
[Openstack Keystone程序结构](https://blog.csdn.net/u010325058/article/details/34845443)
[OpenStack中keystone组件源码流程](https://blog.csdn.net/zhxym/article/details/77374142)
[Keystone controller.py&routers.py代码解析](https://blog.csdn.net/Jmilk/article/details/52067927)

用户alice登录keystone系统（password或者token的方式），获取一个临时的token和catalog服务目录（v3版本登录时，如果没有指定scope，project或者domain，获取的临时token没有任何权限，不能查询project或者catalog）。

alice通过临时token获取自己的所有的project列表。

alice选定一个project，然后指定project重新登录，获取一个正式的token，同时获得服务列表的endpoint，用户选定一个endpoint，在HTTP消息头中携带token，然后发送请求（如果用户知道project name或者project id可以直接第3步登录）。

消息到达endpoint之后，由服务端（nova）的keystone中间件（pipeline中的filter：authtoken）向keystone发送一个验证token的请求。（token类型：uuid需要在keystone验证token，pki类型的token本身是包含用户详细信息的加密串，可以在服务端完成验证）

keystone验证token成功之后，将token对应用户的详细信息，例如：role，username，userid等，返回给服务端（nova）。

服务端（nova）完成请求，例如：创建虚拟机。

服务端返回请求结果给alice。

