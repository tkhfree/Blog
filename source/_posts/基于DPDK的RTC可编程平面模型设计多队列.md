---
title: 基于DPDK的RTC可编程平面模型设计多队列
tags:
  - DPDK
comments: true
toc: true
mathjax: true
categories:
  - Work
abbrlink: 488b1d54
date: 2022-03-15 22:35:49
urlname:
thumbnail:
---

### 基于DPDK的接收报文模式有两种物理实现框架：

- RTC（run to complement）是指所有处理都放在一个cpu上进行，从RX到Filter到TX等过程都由一个进程进行处理
- Pipeline流水线模式是由多个进程进行共同处理，RX在一个进程，Filter在另一个进程，等等，通过环形队列的方式不同进程之间进行报文的传递

PPK平台是在RTC模型框架下进行的开发工作，因此需要针对Qos功能进行新的开发（Pipeline有现有的基于DPDk的Qos处理方式）。

程序处理流程：

```c++
void parse_args(int argc, char **argv){
    //lcore_params[nb_lcore_params].port_id = (uint8_t)int_fld[FLD_PORT];
    //lcore_params[nb_lcore_params].queue_id = (uint8_t)int_fld[FLD_QUEUE];
    //lcore_params[nb_lcore_params].lcore_id = (uint8_t)int_fld[FLD_LCORE];
    ret = parse_lcore_params(optarg);
}
void initialize_args(){
    rte_eal_init();//dpdk 环境抽象层的初始化
    parse_args(argc,argv);//解析传入参数，-l -c -n --lcores之类的
}
void main(){
    initialize_args();
    initialize_nic();
}
```

### DPDK的多线程

DPDK通过在多核设备上创建多线程，每个线程绑定到单独的核上。

在initialize_args里调用DPDK多线程的处理机制，针对每一个网卡port，运行一个新线程，进程分为master线程和slave线程

![](https://pic.imgdb.cn/item/62329dcc5baa1a80ab3f100d.jpg)

DPDK线程基于pthread接口创建，属于抢占式线程模型，受内核调度支配。通过在多核设备上创建多个线程，每个线程绑定到单独的核上，减少线程调度的开销， 以提高性能。控制线程一般绑定到MASTER核上，接受用户配置，并传递配置参数给数据线程等；数据线程分布在不同核上处理数据包。

### 对于网卡开启多队列的流程：

1. 启动时进行端口的配置，这部分需要配置的内容包括设置收包的模式，同时指定收包时使用的哈希函数作用的地方，两者缺一不可；
2. 启动多个队列，这个肯定没问题；
3. 利用DPDK自带的统计工具无法显示相关的队列的数据，这个不知道为什么，需要后续进行源码的阅读后做定夺。
    **所以，测试时只需要从多个队列上进行取数据即可，发现不同队列上都有数据。**



作者：VChao
链接：https://www.jianshu.com/p/c6686b022019
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

 -c 0xff -n 8 -- -p 0x3 --config "\"(0,0,0),(0,1,1),(0,2,2),(1,0,3),(1,1,4),(1,2,5)\""

### 网卡的操作

![接收报文流程](https://pic.imgdb.cn/item/623c87c527f86abb2aabf967.jpg)

智能网卡允许DMA操作和多队列。当配置完多队列的参数后，在启动时，调用initialize_nic();函数，将参数配置到网卡上，网卡开启多队列。DMA由一段连续缓存的环形队列和寄存器构成。（这里引申一下DMA技术，DMA打的实现还要依赖于intel的DDIO，即允许DMA引擎直接操作last level cache，这样cpu就可以尽可能少的参与io操作）寄存器包括base,head,tail,lengh。缓存通常采用一段物理上连续的内存。队列中存放描述符，不管是接收队列还是发送队列，主要有这么几个操作：

1. 填充缓冲区地址（mbuf）到描述符；
2. 移动tail尾指针；
3. 判断描述符中的判断位，确定是否收/发完成。

需要注意的是，环形队列的内容在内存中，但是控制部分寄存器是在网卡上的硬件。

关于NIC、cpu、内存（LLC last level cache）三者一起接收报文的流程：

1. cpu将缓存地址放入环形队列中可用的描述符中；
2. DMA引擎读取描述符获取该缓存地址，将报文放入到该地址的内存（缓存）空间；
3. 网卡回写接收侧描述符，改写描述符状态，确认接收完成的前提下；
4. CPU读取接收侧描述符，确认接收完成。
5. CPU去缓存地址读取报文做转发判断；
6. CPU对报文做填充；准备发送走；

接下来进行发送操作：

1. CPU读取发送侧描述符，确定是否发送（上一条报文）完成；
2. CPU将缓冲区地址写入发送侧描述符；
3. 网卡DMA读取发送侧描述符缓冲地址；
4. 去缓冲地址中取报文数据；
5. 网卡写发送侧描述符，确认发送已经完成；

```c++
uint16_t eth_igb_recv_pkts(void *rx_queue, struct rte_mbuf **rx_pkts, uint16_t nb_pkts)
{
	while (nb_rx < nb_pkts) 
	{
		//从描述符队列中找到待被应用层最后一次接收的那个描述符位置
		rxdp = &rx_ring[rx_id];
		staterr = rxdp->wb.upper.status_error;
		//检查状态是否为dd, 不是则说明驱动还没有把报文放到接收队列，直接退出
		if (! (staterr & rte_cpu_to_le_32(E1000_RXD_STAT_DD)))
		{
			break;
		};
		//找到了描述符的位置，也就从软件队列中找到了mbuf
		rxe = &sw_ring[rx_id];
		rx_id++;
		rxm = rxe->mbuf;		
		//填充mbuf
		pkt_len = (uint16_t) (rte_le_to_cpu_16(rxd.wb.upper.length) - rxq->crc_len);
		rxm->data_off = RTE_PKTMBUF_HEADROOM;
		rxm->nb_segs = 1;
		rxm->pkt_len = pkt_len;
		rxm->data_len = pkt_len;
		rxm->port = rxq->port_id;
		rxm->hash.rss = rxd.wb.lower.hi_dword.rss;
		rxm->vlan_tci = rte_le_to_cpu_16(rxd.wb.upper.vlan);
		//保存到应用层
		rx_pkts[nb_rx++] = rxm;
        
        //申请一个新的mbuf
		nmb = rte_rxmbuf_alloc(rxq->mb_pool);
		//因为原来的mbuf被应用层取走了。这里替换原来的软件队列mbuf，这样网卡收到报文后可以放到这个新的mbuf
		rxe->mbuf = nmb;
		dma_addr = rte_cpu_to_le_64(RTE_MBUF_DATlA_DMA_ADDR_DEFAULT(nmb));
		//将mbuf地址保存到描述符中，相当于高速dma控制器mbuf的地址。
		rxdp->read.hdr_addr = dma_addr;			//这里会将dd标记清0
		rxdp->read.pkt_addr = dma_addr;		
	}
}

uint16_t rte_eth_rx_burst(uint8_t port_id,  uint16_t queue_id, struct rte_mbuf **rx_pkts,  uint16_t nb_pkts)
{
	//如果是e1000网卡，则接收报文的接口为eth_igb_recv_pkts
	return (*dev->rx_pkt_burst)(dev->data->rx_queues[queue_id],	rx_pkts, nb_pkts);
}

```

下图是一个大致的函数实现流程

![](https://pic.imgdb.cn/item/62453e2a27f86abb2af06316.jpg)
