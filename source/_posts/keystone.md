---
title: keystone
tags:
  - OpenStack
comments: true
toc: true
mathjax: true
abbrlink: c202aef8
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---

keystone
User：使用Openstack组件的客户端可以是人、服务、系统，任何的客户端来访问openstack组件，都需要有一个用户
名。
Credentials：是用于确认用户身份的凭证，说白了就是‘信物’，具体可以是:
用户名和密码
用户名和API key
一个 Keystone 分配的身份token
Authentication：
是验证用户身份的过程。Keystone 服务通过检查用户的 Credential 来确定用户的身份。
最开始，使用用户名/密码或者用户名/API key作为credential。当用户的credential被验证后，Kestone 会给用户分配
一个 authentication token 供该用户后续的请求使用。
Keystone中通过Policy（访问规则）来做到基于用户角色(Role)的访问控制。
Token：
是一个数字字符串，访问资源时需要”亮出”你的令牌。在keystone中主要是引入令牌机制来保护用户对于资源的访问，同
时引入PKI（公钥基础实施）对令牌加以保护。
Token包含了在指定范围和有效时间内可以被访问的资源。EG. 在Nova中一个tenant可以是一些虚拟机，在Swift和
Glance中一个tenant可以是一些镜像存储，在Network中一个tenant可以是一些网络资源。
Role：
本质就是一堆ACL的集合，用于划分权限
可以通过给User指定Role，使User获得Role对应的操作权限。
Keystone返回给User的Token包含了Role列表，被访问的Services会判断访问它的User和User提供的Token中所包含的
Role,及每个role访问资源或者进行操作的权限。
系统默认使用管理Role admin和成员Role user（过去的普通用户角色是：member） 。
user验证时必须带有Project(Tenant)
Policy：
对于Keystone service来说，Policy就是一个JSON文件，默认是/etc/keystone/policy.json。通过配置这个文件，
Keystone实现了对User基于Role的权限管理。
OpenStack对User的验证除了OpenStack的身份验证以外，还需要鉴别User对某个Service是否有访问权限。Policy机制
就是用来控制User对Project(Tenant)中资源的操作权限。
Project(Tenant)：
是一个人、或服务所拥有的资源集合。不同的Project之间资源是隔离的，资源可以设置配额。
在一个Project(Tenant)中可以包含多个User，每一个User都会根据权限的划分来使用Project(Tenant)中的资源。比如通
过Nova创建虚拟机时要指定到某个Project中，在Cinder创建卷也要指定到某个Project中。
User访问Project的资源前，必须要与该Project关联，并且指定User在Project下的Role，一个assignment（关联）即：
Project-User-Role
Service：即Openstack中运行的各个组件服务。
Endpoint：
是一个可以通过网络来访问和定位某个Openstack service的地址，通常是一个URL
不同的region有不同的endpoint（我们可以通过endpoint的region属性去定义多个region）。
当Nova需要访问Glance服务去获取image 时，Nova通过访问Keystone拿到Glance的endpoint，然后通过访问该
endpoint去获取Glance服务。
Endpoint 分为三类：

admin url –> 给admin用户使用，Port：35357
internal url –> OpenStack内部服务使用来跟别的服务通信，Port：5000
public url –> 互联网用户可以访问的地址，Port：5000
Catalog：
用户和服务可以使用使用keystone管理的catalog，定位到其他的服务，catalog一个openstack部署的相关服务的集
合，每个服务都有一个或者多个endpoint（即可以访问的url地址），即catalog=services+endpoint。每个endpoint
可以分为三种类型：
admin，internal，public，在生产环境中，不同endpoint类型位于不同的网络来为不同的用户使用（提高安全性），比
如:
public API:对整个互联网可见，这样客户就可以方便的管理自己的云了。
admin API:应该严格限定只有管理云基础设施的组织内的运营商，才能使用该API
internel API:应该被限定只有那些安装有OpenStack服务的主机，才能使用该API
Service与Endpoint关系介绍：
在openstack中，每一个service都有三种endpoint. Admin, public, internal（创建完service后需要为其创建API
EndPoint. ）
Admin是用作管理用途的，如它能够修改user/tenant(project)。
public 是让客户调用的，比如可以部署在外网上让客户可以管理自己的云。
internal是openstack内部调用的。
三种endpoints 在网络上开放的权限一般也不同。Admin通常只能对内网开放，public通常可以对外网开放，internal通
常只能对安装有openstack对服务的机器开放。
endpoint举例
Regions：
openstack支持多个可扩展的regions，OpenStack的支持可扩展的多个区域。为简单起见，一般使用管理网络ip地址作
为所有endpoint类型(三种api)的ip，且所有的endpoint类型(三种api)都使用一个区域，即regionone区。
每个你部署的openstack服务都需要绑定endpoint（存储在keystone中）来提供服一个服务的入口，因而我们第一需要
部署的组件就是keystone。
V3新增的概念：
Tenant 重命名为 Project
添加了 Domain 的概念
添加了 Group 的概念

