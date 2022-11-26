---
title: k8s安装
tags:
  - Kubernetes
comments: true
toc: true
mathjax: true
abbrlink: 1551c243
date: 2018-11-18 00:21:52
urlname:
categories:
thumbnail:
---

k8s集群解决的问题
使用：
先使用yaml文件（专门编写配置文件的一种格式）创建rc（定义标签，replicas数量，保存的容器模版等），自动会起按
配置规定的pod，之后配置service文件（定义标签，端口等），并且起服务的时候，k8s会自动把ip地址写入linux的环境变量，并且自动在容器内部生成相关的环境变量。
之所以将多个容器封在pod内，因为容器间通信很麻烦，pod相当与一个应用层的逻辑宿主机，pod内通信可以使用localhost。
k8s中pod rc service node皆视为对象，label可以以key/value对形式附加上去。
labels：
key1:value1
key2:value2
通过Kubectl提交一个创建RC的请求，请求通过API Server被写入etcd中。
Controller Manager通过API Server的监听资源变化的接口监听到这个RC事件，分析之后，发现当前集群中还没有它所
对应的Pod实例，于是根据RC里的Pod模板定义生成一个Pod对象，通过API Server写入etcd。
Scheduler发现后，执行调度流程，为这个新Pod选定一个落户的Node，然后通过API Server将结果写入到etcd中。
随后，目标Node上运行的Kubelet进程通过API Server监测到这个“新生的”Pod，并按照它的定义，启动该Pod直到Pod
的生命结束。
接着，用户通过Kubectl提交一个新的映射到该Pod的Service的创建请求，Controller Manager会通过Label标签查询到
相关联的Pod实例，生成Service的Endpoints信息，并通过API Server写入到etcd中，接下来，所有Node上运行的
Proxy进程通过API Server查询并监听Service对象与其对应的Endpoints信息，建立一个软件方式的负载均衡器来实现
Service访问到后端Pod的流量转发功能。
各个进程的作用如下：
etcd 用于持久化存储集群中所有的资源对象，如Node、Service、Pod、RC、Namespace等；API Server提供了操作
etcd的封装接口API，这些API基本上都是集群中资源对象的增删改查及监听资源变化的接口。
API Server 提供了资源对象的唯一操作入口，其他所有组件都必须通过它提供的API来操作资源数据，通过对相关的资源
数据“全量查询”+“变化监听”，这些组件可以很“实时”地完成相关的业务功能。
Controller Manager 集群内部的管理控制中心，其主要目的是实现Kubernetes集群的故障检测和恢复的自动化工作，
比如根据RC的定义完成Pod的复制或移除，以确保Pod实例数符合RC副本的定义；根据Service与Pod的管理关系，完成
服务的Endpoints对象的创建和更新；其他诸如Node的发现、管理和状态监控、死亡容器所占磁盘空间及本地缓存的镜
像文件的清理等工作也是由Controller Manager完成的。
Scheduler 集群中的调度器，负责Pod在集群节点中的调度分配。

Kubelet 负责本Node节点上的Pod的创建、修改、监控、删除等全生命周期管理，同时Kubelet定时“上报”本Node的状
态信息到API Server里。
Proxy 实现了Service的代理与软件模式的负载均衡器。

1. Docker之间跨节点的通讯
2. 动态管理集群负载，使集群工作在期望的状态
3. 集群之间资源的调度
4. 集群的运行方式

k8s的节点类型
1. master节点负责整个集群的控制和管理
2. node节点是负载节点，运行pod。

master节点所用到的组件
1. kube-apiserver提供整个集群资源的操作入口（增删改查），也是集群的控制入口，提供http rest接口。
2. etcd保存整个集群的状态，也即是资源信息。KEY/VALUE模式的存储系统
etcd的写入是对于每个对象进行的，例如创建rc、pod、namespace、监听pod变化，调度pod到node节点，其他
各个工具都是通过api server写入etcd或者通过api server监听
3. kube-schedule负载pod（资源）的调度，根据设置策略调度pod到指定的node上运行。
4. kube-controller-manager维护集群，所有资源的自动化控制中心。当整个集群的状态与期望不符合的时候，组件
会努力让集群恢复期望状态，比如：当一个pod死亡，组件新建一个pod恢复对应replicas set期望的状态。

node节点所用到的组件
1. kubelet负责pod的生命周期管理，同时与master密切协作，实现集群管理的基本功能
2. kube-proxy实现service的内部服务发现与负载均衡机制的重要组件，主要通过设置iptables规则实现
3. docker engin负责docker的生命周期管理

命名空间
k8s的namespace是在将系统内部的对象“分配”到不同的空间下，形成逻辑上的分组，便于不同的分组在共享使用整个集
群的资源同时还能被分别管理。
POD RC Service Deployment等资源都会运行在特定的namespace中，默认的namerspace是default。

resource quotas
空白

lable
通过lable进行弱关联，灵活的分类或者选择不同服务或者业务，可以采用松耦合方式进行服务部署。label是一对
key/value（想到etcd也是一对KEY/VALUE模式的存储系统

），对k8s没有什么意义，但是对于用户意义很大

replication controller（RC）
RC是K8S另一种核心概念，应用托管到K8s后，他会确保任何时间K8S都有指定数量的Pod
同一个label的pod，高可用（有备用）
service同时保证有同一个固定ip

Deployment
deployment同样是为了确保pod的数量和健康，90%与RC一样。

service
kubernetes中service是一种抽象的概念，它定义了一个pod的逻辑集合以及访问他们的策略，service同pod的关联是居
于label完成。service的目标是提供一种桥梁，它会为访问者提供一个固定的ip，用于在访问时重定向到相应的后端，使
得非K8s原生应用程序，在无需为kubemces编写特定代码的前提下，轻松访问。
pod是k8s的最小管理资源，一群pod被rc或者deployment高可用，pod里包含相关的一组容器（完整的提供一个服
务），pod有一个pause的容器，其他容器是业务容器，共享pause容器的网络栈和volume挂载卷。如果该pod对内或者
对外提供服务，则可以映射到一个service上。

volume
容器内数据随着容器的生命周期而存在，如果需要持久化的保存数据，则需要挂载数据卷
1. emptyDir
2. hostPath(local)
3. NFS
4. glusterFS
5. cephFS

网络通信场景
1. 容器到容器之间的直接通信
2. POD到POD之间的通信（同一节点/不同节点）
3. POD到service之间的通信
4. 集群外部与内部组件之间的通信
安装
安装准备
•操作系统：CentOS7.3
[root@centos7-base-ok]# cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
•安装机器：k8s-1为master节点，k8s-2、k8s-3为slave节点
[root@centos7-base-ok]# cat /etc/hosts
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
k8s-1 192.168.80.28

k8s-2 192.168.80.35
k8s-3 192.168.80.14
安装步骤
安装docker 1.12(所有节点)
注意：现在docker已经更新到CE版本，但是kubernetes官方文档说在1.12上测试通过，最近版本的兼容性未测试，为
了避免后面出现大坑，我们还是乖乖安装1.12版本的docker。
1.新建docker.repo文件，将文件移动到/etc/yum.repos.d/目录下
[root@centos7-base-ok]# cat /etc/yum.repos.d/docker.repo
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
2.运行yum命令，找到需要安装的docker版本
10:21 [root@centos7-base-ok]# yum list|grep docker | sort -r
python2-avocado-plugins-runner-docker.noarch
python-dockerpty.noarch 0.4.1-6.el7 epel
python-dockerfile-parse.noarch 0.0.5-1.el7 epel
python-docker-scripts.noarch 0.4.4-1.el7 epel
python-docker-pycreds.noarch 1.10.6-1.el7 extras
python-docker-py.noarch 1.10.6-1.el7 extras
kdocker.x86_64 4.9-1.el7 epel
golang-github-fsouza-go-dockerclient-devel.x86_64
docker.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-v1.10-migrator.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-unit-test.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-registry.x86_64 0.9.1-7.el7 extras
docker-registry.noarch 0.6.8-8.el7 extras
docker-python.x86_64 1.4.0-115.el7 extras
docker-novolume-plugin.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-lvm-plugin.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-logrotate.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-latest.x86_64 1.13.1-13.gitb303bf6.el7.centos
docker-latest-v1.10-migrator.x86_64 1.13.1-13.gitb303bf6.el7.centos
docker-latest-logrotate.x86_64 1.13.1-13.gitb303bf6.el7.centos
docker-forward-journald.x86_64 1.10.3-44.el7.centos extras
docker-engine.x86_64 17.05.0.ce-1.el7.centos dockerrepo
docker-engine.x86_64 1.12.6-1.el7.centos @dockerrepo
docker-engine-selinux.noarch 17.05.0.ce-1.el7.centos @dockerrepo
docker-engine-debuginfo.x86_64 17.05.0.ce-1.el7.centos dockerrepo
docker-distribution.x86_64 2.6.1-1.el7 extras
docker-devel.x86_64 1.3.2-4.el7.centos extras
docker-compose.noarch 1.9.0-5.el7 epel
docker-common.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-client.x86_64 2:1.12.6-32.git88a4867.el7.centos
docker-client-latest.x86_64 1.13.1-13.gitb303bf6.el7.centos
cockpit-docker.x86_64 141-3.el7.centos extras

3.找到对应版本后，执行yum install -y 包名＋版本号，安装1.12版本的docker-engine
[root@centos7-base-ok]# yum install -y docker-engine.x86_64-1.12.6-1.el7.centos
4.执行docker version命令，验证docker安装版本，执行docker run命令，验证docker是否安装成功
[root@centos7-base-ok]# docker version
Client:
Version: 1.12.6
API version: 1.24
Go version: go1.6.4
Git commit: 78d1802
Built: Tue Jan 10 20:20:01 2017
OS/Arch: linux/amd64
Server:
Version: 1.12.6
API version: 1.24
Go version: go1.6.4
Git commit: 78d1802
Built: Tue Jan 10 20:20:01 2017
OS/Arch: linux/amd64
5.设置开机启动，启动容器，docker安装完成
[root@centos7-base-ok]# systemctl enbale docker && systemctl start docker
安装kubectl、kubelet、kubeadm(根据需求在不同节点安装)
注意：此步骤是填坑的开始，因为官方文档的yum源在国内无法使用，安装完成后注意观察你的/var/log/message日
志，会疯狂报错，别着急，跟着我一步一步来填坑。
1.新建kubernetes.repo文件，将文件移动到/etc/yum.repos.d/目录下(所有节点)
[root@centos7-base-ok]# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
2.通过yum安装kubectl、kubelet、kubeadm(所有节点)
[root@centos7-base-ok]# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
3.修改kubelet配置，启动kubelet(所有节点)
注意：时刻查看/var/log/message的日志输出，会看到kubelet一直启动失败。
编辑10-kubeadm.conf的文件，修改cgroup-driver配置：
[root@centos7-base-ok]# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
[Service]
Environment=”KUBELET_KUBECONFIG_ARGS=–kubeconfig=/etc/kubernetes/kubelet.conf –requirekubeconfig=true”

Environment=”KUBELET_SYSTEM_PODS_ARGS=–pod-manifest-path=/etc/kubernetes/manifests –allowprivileged=true”
Environment=”KUBELET_NETWORK_ARGS=–network-plugin=cni –cni-conf-dir=/etc/cni/net.d –cni-bindir=/opt/cni/bin”
Environment=”KUBELET_DNS_ARGS=–cluster-dns=10.96.0.10 –cluster-domain=cluster.local”
Environment=”KUBELET_AUTHZ_ARGS=–authorization-mode=Webhook –client-cafile=/etc/kubernetes/pki/ca.crt”
Environment=”KUBELET_CADVISOR_ARGS=–cadvisor-port=0”
Environment=”KUBELET_CGROUP_ARGS=–cgroup-driver=cgroupfs”
ExecStart=
ExecStart=/usr/bin/kubelet
KU BELE TN ET W ORKARGS
KU BELE TC GROU PARGS

KU BELE TK U BECON F I GARGS

KUBELET_DNS_ARGS

KUBELET_SYSTEM_PODS_ARGS

KU BELE TAU T H ZARGS

KUBELET_CADVISOR_ARGS

KUBELET_EXTRA_ARGS

将“–cgroup-driver=systems”修改成为“–cgroup-driver=cgroupfs”，重新启动kubelet。
[root@centos7-base-ok]# systemctl restart kubelet
4.下载安装k8s依赖镜像
注意：此步骤非常关键，kubenetes初始化启动会依赖这些镜像，天朝的网络肯定是拉不下来google的镜像的，一般人
过了上一关，这一关未必过的去，一定要提前把镜像下载到本地，kubeadm安装才会继续，下面我会列出来master节点
和node依赖的镜像列表。（备注：考虑到随着kubernetes版本一直更新，镜像也可能会有变化，大家可以先执行
kubeadm init 生成配置文件，日志输出到 [apiclient] Created API client, waiting for the control plane to become
ready 这一行就会卡住不动了，你可以直接执行 ctrl + c 中止命令执行，然后查看 ls -ltr
/etc/kubernetes/manifests/
yaml文件列表，每个文件都会写着镜像的地址和版本）
在这里我提一个可以解决下载google镜像的方法，就是买一台可以下载的机器，安装代理软件，在需要下载google镜像
的机器的docker设置 HTTP_PROXY 配置项，配置好自己的服务代理即可（也可以直接买可以访问到google的服务器安
装).
master节点：
REPOSITORY TAG IMAGE ID CREATED SIZE
quay.io/calico/kube-policy-controller v0.7.0 fe3174230993 3 days ago 21.94 MB
kubernetesdashboarddev/kubernetes-dashboard-amd64 head e2cadb73b2df 5 days ago 136.5 MB
quay.io/calico/node v2.4.1 7643422fdf0f 6 days ago 277.4 MB
gcr.io/google_containers/kube-controller-manager-amd64 v1.7.3 d014f402b272 11 days ago 138 MB
gcr.io/google_containers/kube-apiserver-amd64 v1.7.3 a1cc3a3d8d0d 11 days ago 186.1 MB
gcr.io/google_containers/kube-scheduler-amd64 v1.7.3 51967bf607d3 11 days ago 77.2 MB
gcr.io/google_containers/kube-proxy-amd64 v1.7.3 54d2a8698e3c 11 days ago 114.7 MB
quay.io/calico/cni v1.10.0 88ca805c8ddd 13 days ago 70.25 MB
gcr.io/google_containers/kubernetes-dashboard-amd64 v1.6.3 691a82db1ecd 2 weeks ago 139 MB
quay.io/coreos/etcd v3.1.10 47bb9dd99916 4 weeks ago 34.56 MB
gcr.io/google_containers/k8s-dns-sidecar-amd64 1.14.4 38bac66034a6 7 weeks ago 41.81 MB
gcr.io/google_containers/k8s-dns-kube-dns-amd64 1.14.4 a8e00546bcf3 7 weeks ago 49.38 MB
gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64 1.14.4 f7f45b9cb733 7 weeks ago 41.41 MB
gcr.io/google_containers/etcd-amd64 3.0.17 243830dae7dd 5 months ago 168.9 MB
gcr.io/google_containers/pause-amd64 3.0 99e59f495ffa 15 months ago 746.9 kB
node节点：
[root@centos7-base-ok]# docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
kubernetesdashboarddev/kubernetes-dashboard-amd64 head e2cadb73b2df 5 days ago 137MB
quay.io/calico/node v2.4.1 7643422fdf0f 6 days ago 277MB
gcr.io/google_containers/kube-proxy-amd64 v1.7.3 54d2a8698e3c 11 days ago 115MB

gcr.io/google_containers/kube-proxy-amd64 v1.7.3 54d2a8698e3c 11 days ago 115MB
quay.io/calico/cni v1.10.0 88ca805c8ddd 13 days ago 70.3MB
gcr.io/google_containers/kubernetes-dashboard-amd64 v1.6.3 691a82db1ecd 2 weeks ago 139MB
nginx latest b8efb18f159b 2 weeks ago 107MB
hello-world latest 1815c82652c0 2 months ago 1.84kB
gcr.io/google_containers/pause-amd64 3.0 99e59f495ffa 15 months ago 747kB
5.利用kubeadm初始化服务(master节点)
注意：如果你在上一步执行过 kubeadm init 命令，没有关系，此步执行只需要执行时加上 –skip-preflight-checks 这
个配置项即可。
注意：执行 kubeadm init 的 –pod-network-cidr 参数和选择的网络组件有关系，详细可以看官方文档说明,本文选用
的网络组件为 Calico
[root@centos7-base-ok]# kubeadm init –pod-network-cidr=192.168.0.0/16 –apiserver-advertiseaddress=0.0.0.0 –apiserver-cert-extra-sans=192.168.80.28,192.168.80.14,192.168.80.35,127.0.0.1,k8s1,k8s-2,k8s-3,192.168.0.1 –skip-preflight-checks
参数说明：
参数名称
必选
参数说明
pod-network-cidr Yes For certain networking solutions the Kubernetes master can also play a role in
allocating network ranges (CIDRs) to each node. This includes many cloud providers and flannel. You can
specify a subnet range that will be broken down and handed out to each node with the –pod-network-cidr
flag. This should be a minimum of a /16 so controller-manager is able to assign /24 subnets to each node
in the cluster. If you are using flannel with this manifest you should use –pod-network-cidr=10.244.0.0/16.
Most CNI based networking solutions do not require this flag.
apiserver-advertise-address Yes This is the address the API Server will advertise to other members of the
cluster. This is also the address used to construct the suggested kubeadm join line at the end of the init
process. If not set (or set to 0.0.0.0) then IP for the default interface will be used.
apiserver-cert-extra-sans Yes Additional hostnames or IP addresses that should be added to the Subject
Alternate Name section for the certificate that the API Server will use. If you expose the API Server through a
load balancer and public DNS you could specify this with.
其它的 kubeadm 参数设置请参照 官方文档
6.做一枚安静的美男子，等待安装成功，安装成功后你会看到日志如下(master节点):
注意：记录这段日志，后面添加node节点要用到。
[apiclient] All control plane components are healthy after 22.003243 seconds
[token] Using token: 33729e.977f7b5d0a9b5f3e
[apiconfig] Created RBAC rules
[addons] Applied essential addon: kube-proxy
[addons] Applied essential addon: kube-dns
Your Kubernetes master has initialized successfully!
To start using your cluster, you need to run (as a regular user):
mkdir -p

H OM E/. kubesudocp − i/etc/kubernetes/admin. conf

HOME/.kube/config

sudo chown

(id − u) :

(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run “kubectl apply -f [podnetwork].yaml” with one of the options listed at:
http://kubernetes.io/docs/admin/addons/
You can now join any number of machines by running the following on each node
as root:
kubeadm join –token xxxxxxx 192.168.80.28:6443
7.创建kube的目录，添加kubectl配置(master节点)
mkdir -p

H OM E/. kubesudocp − i/etc/kubernetes/admin. conf

sudo chown

(id − u) :

HOME/.kube/config

(id -g) $HOME/.kube/config

8.用 kubectl 添加网络组件Calico(master节点)
kubectl apply -f http://docs.projectcalico.org/v2.4/gettingstarted/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
注意：此处坑为该文件未必下载的到，建议还是提前下载到本地，然后执行 kubectl apply -f <本地路径>
9.确认安装是否成功(master节点)
9.1 打开你的/var/log/messages，查看是否有报错，理论上，执行完上一步过去5分钟，日志应该不会有任何错误出
现，如果持续报错，并且过了10分钟错误依然没有消失，检查之前的步骤是否有问题
9.2 运行 kubectl get pods –all-namespaces 查看结果,如果STATUS都为Running，恭喜你，你的master已经安装成
功了。
注意：你的结果显示的条数未必和我完全一样，因为我这里有node节点的相关信息，而你还没有添加node节点。
[root@centos7-base-ok]# kubectl get pods –all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
default nginx-app-1666850838-4z2tb 1/1 Running 0 3d
kube-system calico-etcd-0ssdd 1/1 Running 0 3d
kube-system calico-node-1zfxd 2/2 Running 1 3d
kube-system calico-node-s2gfs 2/2 Running 1 3d
kube-system calico-node-xx30v 2/2 Running 1 3d
kube-system calico-policy-controller-336633499-wgl8j 1/1 Running 0 3d
kube-system etcd-k8s-1 1/1 Running 0 3d
kube-system kube-apiserver-k8s-1 1/1 Running 0 3d
kube-system kube-controller-manager-k8s-1 1/1 Running 0 3d
kube-system kube-dns-2425271678-trmxx 3/3 Running 1 3d
kube-system kube-proxy-79kkh 1/1 Running 0 3d
kube-system kube-proxy-n1g6j 1/1 Running 0 3d
kube-system kube-proxy-vccr6 1/1 Running 0 3d
kube-system kube-scheduler-k8s-1 1/1 Running 0 3d
10.安装node节点,执行在master节点执行成功输出的日志语句(node节点执行)
注意：执行如下语句的之前，一定要确认node节点下载了上文提到的镜像，否则因为镜像下载不成功会导致node节点初
始化失败；第二点，一定要时刻查看/var/log/messages日志，如果镜像版本发生变化，在日志里会提示需要下载的镜
像；第三点，就是要有耐心，如果你的网络可以下载到镜像，你当个安静的美男子就可以了，因为 kubeadm 会帮你做
一切，知道你发现/var/log/messages不再有错误日志出现，说明它已经帮你搞定了所有事情，你可以开心的玩耍了。
[root@centos7-base-ok]# kubeadm join –token xxxxxxxx 192.168.80.28:6443
1.验证子节点，在master节点执行 kubectl get nodes 查看节点状态。

注意：node的状态会变化，添加成功后才是Ready。
[root@centos7-base-ok]# kubectl get nodes
NAME STATUS AGE VERSION
k8s-1 Ready 3d v1.7.3
k8s-2 Ready 3d v1.7.3
k8s-3 Ready 3d v1.7.3
12.恭喜你，你可以开心的进行kubernetes1.7.3之旅了
安装后记
Kubernetes，想说爱你不容易啊 ,欢迎其它团队或者个人与我们团队进行交流，有意向可以评论区给我留言。
补充：目前官方说dashboard的HEAD版本支持1.7，但是我试了下dashboard确实不行，希望官方加快修复，还有就是
多些错误定位的方法，否则很难提出具体的问题。

