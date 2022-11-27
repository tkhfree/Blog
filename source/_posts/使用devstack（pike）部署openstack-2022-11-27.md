---

title: 使用devstack（pike）部署openstack
date: 2022-11-27 12:00
tags:
  -  openstack
  -  python

categories:
  - 方法：方法论（经验谈）

comments: true
toc: true

---
[toc]



# 环境和工具

ubuntu16.04 + devstack（pike）

- 更改apt/或者yum源

- 更改pip源

- 提前下载好glance、neutron、nova、keystone、cinder等镜像，放在files文件夹下

- git clone devstack

- 创建stack用户，赋予用户权限

- ./stack.sh 安装不成功./unstack.sh ./clean.sh

  

  **<font color=yellow>the most beautiful thing is not keep time but retain memories.</font>**

参考[陈沙克](http://www.chenshake.com/local-conf-devstack-profile-parameter-description/)大佬说明解释 Local.conf



### 开始



### 修改git源

默认Devstack会从github下载所有需要的代码，包括OpenStack。这其实是导致Devstack安装时间太长的一个重要原因。

目前 git.trystack.cn 提供OpenStack的所有github的mirror。对于Devstack来说，只需要在配置文件增加3行就可以。

```shell
# use TryStack git mirror
GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git
```

### 主机IP

```shell
HOST_IP=192.168.27.128
```

### 镜像下载

安装devstack的时候，默认会下载相应的镜像，这些镜像都在国外，我们可以指定连接来下载相关镜像。

```shell
# Define images to be automatically downloaded during the DevStack built process.
DOWNLOAD_DEFAULT_IMAGES=False
IMAGE_URLS=http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

### ipv4

```shell
# only support IP v4
SERVICE_IP_VERSION=4
```

### keystone 版本

现在默认就是支持keystone v3，除非你需要使用v2。

```shell
# only support keystone v2
ENABLE_IDENTITY_V2=True
```

### 网络

默认Devstack会创建一个网络，如果你不需要创建

```shell
#not create default network
NEUTRON_CREATE_INITIAL_NETWORKS=False
```

我们也可以指定相关的网络

```shell
# instead of default network
FLOATING_RANGE="192.168.27.0/24"
FIXED_RANGE="10.0.0.0/24"
Q_FLOATING_ALLOCATION_POOL=start=192.168.27.102,end=192.168.27.110
PUBLIC_NETWORK_GATEWAY="192.168.27.2"
```

### 指定版本安装

对于普通用户，想了解某个版本的功能，可以在配置文件指定版本

```shell
# Branches
KEYSTONE_BRANCH=stable/liberty
NOVA_BRANCH=stable/liberty
NEUTRON_BRANCH=stable/liberty
SWIFT_BRANCH=stable/liberty
GLANCE_BRANCH=stable/liberty
CINDER_BRANCH=stable/liberty
```

我们使用的Devstack，也需要使用相同的版本，这样才能避免安装失败可能性。

```shell
git clone http://git.trystack.cn/openstack-dev/devstack -b stable/liberty
```

默认大家都是使用devstack的master。

### Neutron

这是最复杂的地方，目前devstack默认的网络还是nova network，所以你要采用Neutron，你必须

```shell
# Enabling Neutron (network) Service
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service q-metering
enable_service neutron
```

vlan模式

```shell
# VLAN configuration.
Q_PLUGIN=ml2
ENABLE_TENANT_VLANS=True
TENANT_VLAN_RANGE=1100:2999
```

### 离线安装

当我们修改参数，重新运行devstack的时候，这个时候，你不希望重新下载git和操作系统的update

```shell
# Work offline
#OFFLINE=True
# Reclone each time
RECLONE=no
```

# 安装准备

1. apt更改国内源并更新
2. 安装pip并升级
3. 更改pip源地址为国内
4. 安装git工具
5. 安装Open vSwitch

# 下载源包

```shell
git clone https://github.com/openstack-dev/devstack.git
```

目前Devstack脚本已经不支持直接使用root身份运行，你需要创建stack用户运行

```
cd /home/devstack/tools/
./create-stack-user.sh
```

修改devstack目录权限,让stack用户可以运行

```
chown -R stack:stack /home/devstack
```

切换的stack用户下

```
su stack
cd /home/devstack
```

进入devstack目录下，创建local.conf 文件

```shell
# Misc
DATABASE_PASSWORD=123456
ADMIN_PASSWORD=123456
SERVICE_PASSWORD=123456
SERVICE_TOKEN=123456
RABBIT_PASSWORD=123456
 
# Reclone each time
RECLONE=true
  
# Python enviroments
#OFFLINE=true
  
## For Keystone
KEYSTONE_TOKEN_FORMAT=PKI
  
## For Swift
#SWIFT_REPLICAS=1
#SWIFT_HASH=011688b44136573e209e
  
# Enable Logging
DEST=/home/stack
LOGFILE=$DEST/logs/stack.sh.log
VERBOSE=True
LOG_COLOR=True
SCREEN_LOGDIR=$DEST/logs
  
# Pre-requisite
ENABLED_SERVICES=rabbit,mysql,key
  
## If you want ZeroMQ instead of RabbitMQ (don't forget to un-declare 'rabbit' from the pre-requesite)
#ENABLED_SERVICES+=,-rabbit,-qpid,zeromq
  
## If you want Qpid instead of RabbitMQ (don't forget to un-declare 'rabbit' from the pre-requesite)
#ENABLED_SERVICES+=,-rabbit,-zeromq,qpid
  
# Horizon (Dashboard UI) - (always use the trunk)
ENABLED_SERVICES+=,horizon
#HORIZON_REPO=https://github.com/openstack/horizon
#HORIZON_BRANCH=master
  
# Nova - Compute Service
ENABLED_SERVICES+=,n-api,n-crt,n-obj,n-cpu,n-cond,n-sch,n-novnc,n-cauth
IMAGE_URLS+=",https://launchpad.net/cirros/trunk/0.3.0/+download/cirros-0.3.0-x86_64-disk.img"
  
## Nova Cells
ENABLED_SERVICES+=,n-cell
  
# Glance - Image Service
ENABLED_SERVICES+=,g-api,g-reg
  
# Swift - Object Storage
#ENABLED_SERVICES+=,s-proxy,s-object,s-container,s-account
  
# Neutron - Networking Service
# If Neutron is not declared the old good nova-network will be used
ENABLED_SERVICES+=,q-svc,q-agt,q-dhcp,q-l3,q-meta,neutron
  
## Neutron - Load Balancing
ENABLED_SERVICES+=,q-lbaas
  
## Neutron - VPN as a Service
ENABLED_SERVICES+=,q-vpn
  
## Neutron - Firewall as a Service
ENABLED_SERVICES+=,q-fwaas
  
# VLAN configuration - LinuxBridge + VLAN模式
Q_PLUGIN=linuxbridge
ENABLE_TENANT_VLANS=True
TENANT_VLAN_RANGE=1920:1930
PHYSICAL_NETWORK=default
LB_PHYSICAL_INTERFACE=eth0
  # VLAN configuration - Open VSwitch + VLAN模式  
  #ENABLE_TENANT_VLANS=True
  #TENANT_VLAN_RANGE=1920:1930
  #PHYSICAL_NETWORK=default
  #OVS_PHYSICAL_INTERFACE=eth0

# GRE tunnel configuration
#Q_PLUGIN=ml2
#ENABLE_TENANT_TUNNELS=True
  
# VXLAN tunnel configuration
#Q_PLUGIN=ml2
#Q_ML2_TENANT_NETWORK_TYPE=vxlan
  
# Cinder - Block Device Service
ENABLED_SERVICES+=,cinder,c-api,c-vol,c-sch
  
# Heat - Orchestration Service
ENABLED_SERVICES+=,heat,h-api,h-api-cfn,h-api-cw,h-eng
#IMAGE_URLS+=",http://fedorapeople.org/groups/heat/prebuilt-jeos-images/F17-x86_64-cfntools.qcow2"
  
# Ceilometer - Metering Service (metering + alarming)
CEILOMETER_BACKEND=mysql
ENABLED_SERVICES+=,ceilometer-acompute,ceilometer-acentral,ceilometer-collector,ceilometer-api
ENABLED_SERVICES+=,ceilometer-alarm-notify,ceilometer-alarm-eval
  
# Apache fronted for WSGI
APACHE_ENABLED_SERVICES+=keystone,swift
```

# 预防问题

1. 手动下载各个组件的的包，解压至/opt/stack下并使用git工具初始化各个组件的仓库

2. 手动下载requirement包，并手动安装依赖包

   ```shell
   pip install -r upper-requirement.txt
   ```

3. 一定要升级pip工具，如果遇到setuptools、get-pip.py之类的报错，基本就是pip安装与easy_install安装这两种python包管理工具的冲突，使用pip升级setuptool、disturb

4. 安装过程中比较难下的还有etcd-v3.1.7，stestr等，都可以提前下好

5. 如果报缺少openstack_auth，重写安装dajago_openstack_auth

6. 缺少model_query，全局搜索_model_query，复制到/usr/local/bin

7. 要安装p4插件，提前下好heat、heat-dashboard、networking-sfc、networking-p4

8. p4的底层驱动behavioral-model里有一个bvm2.py的脚本，脚本里有一个在临时文件夹下下载thrift的命令，可以手动下载，同时脚本的thrift0.7.0版本貌似不兼容，下载thrift0.10.0可以运行

9. 如果keystone日志报uwsgi没有启动，删除/etc/keystone/***.ini脚本里plugins=python

10. 控制节点执行：nova-manage cell_v2 discover_hosts --verbose以发现计算节点