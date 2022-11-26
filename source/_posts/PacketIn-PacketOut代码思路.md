---
title: PacketIn/PacketOut代码思路
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - null
abbrlink: 8211506c
date: 2020-09-09 16:53:07
urlname:
categories:
---

packet-in与packet-out是控制平面与数据平面进行流交互的工具，因为不同于下流表或者交互配置信息等方式，packet-in/out具有流形式和任意一端都可主动发起请求的特点，因此采用grpc的异步双向流交互进行通讯的工具。

![](https://pic.downk.cc/item/5fd9571a3ffa7d37b3b5a980.gif)

业务逻辑上如下：

## Packet-In

使用Packet-In消息的目的是为了将到达OpenFlow交换机的数据包发送至OpenFlow控制器。以下2种情况即可发送Packet-In消息。

> 不存在与流表项一致的项目时（Table-miss），OFPR_NO_MATCH
> 匹配的流表项中记载的行动为“发送至OpenFlow控制器”时，OFPR_ACTION

发送Packet-In消息时OpenFlow交换机分为两种情况，一种是缓存数据包，一种是不缓存数据包。如果不通过OpenFlow交换机缓存数据包，那么Packet-In消息的buffer_id字段设置为-1，将整个数据包发送至OpenFlow控制器。
如果通过OpenFlow交换机缓存数据包，那么以通过SET_CONFIG消息设置的miss_send_len为最大值的数据包数据将发送至OpenFlow控制器。
miss_send_len的默认值为128。未实施SET_CONFIG消息的交换时，使用该默认值。

|           |        |                                          |
| :-------- | :----- | :--------------------------------------- |
| 字段      | 比特数 | 内容                                     |
| buffer_id | 32     | 表示OpenFlow交换机中保存的数据包的缓存id |
| Total_len | 16     | 帧的长度                                 |
| in_port   | 16     | 接受帧的端口                             |
| reason    | 8      | 发送Packet-in消息的原因                  |
| pad       | 8      | 用于调整对齐的填充                       |
| data      | 任意   | 包含以太网帧的数据时使用的字段。         |

控制器通过grpc与交换机进行交互，因此在使用proto生成runtime库之后，在server端导入P4runtime库函数（p4runtime.pb.h、p4runtime.pb.cc、p4runtime.grpc.pb.h、p4runtime.grpc.pb.cc）后，启动监听ip和端口，注册proto文件中的rpc服务，等待stub的调用。

## Packet-out

Packet-Out消息是从OpenFlow控制器向OpenFlow交换机发送的消息，是包含数据包发送命令的消息”。
若OpenFlow交换机的缓存中已存在数据包，而OpenFlow控制器发出“发送该数据包”的命令时，该消息指定了表示相应数据包的buffer_id。使用Packet-Out消息还可将OpenFlow控制器创建的数据包发送至OpenFlow交换机。此时，buffer_id置为-1，在Packet-Out消息的最后添加数据包数据。

![](https://pic.downk.cc/item/5f58a602160a154a67071313.jpg)

| 字段        | 比特数 | 内容                                     |
| ----------- | ------ | ---------------------------------------- |
| buffer_id   | 32     | 表示OpenFlow交换机中保存的数据包的缓存id |
| in_port     | 16     | 数据包的输入端口                         |
| actions_len | 16     | 行动信息的长度                           |

## 流程

控制器与OpenFlow交换机在连接建立过程中会存在拓扑发现的环节，该环节会密集出现Packe-in/out消息，其交互流程如下：

![](https://pic.downk.cc/item/5f58a655160a154a6707714c.jpg)

1、 [SDN控制器](https://www.baidu.com/s?wd=SDN控制器&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)通过构造Packet-out消息，向交换机s1的三个端口分别发送上图所示的[LLDP](https://baike.baidu.com/item/LLDP/6849522?fr=aladdin)数据包。
2、 控制器向交换机s1下发流表，流表规则为：将从Controller端口收到的LLDP数据包从规定端口发送出去。
3、 控制器向交换机s2下发流表，流表规则为：将从非Controller接收到LLDP数据包发送给控制器。
4、 当LLDP数据包到达交换机s2，会触发Packet-in消息发往控制器。控制器通过解析LLDP数据包，得到链路的源交换机，源接口（s1,port1）。通过收到的Packet-in消息知道目的交换机(s2)。
5、 同理，当SDN控制器向交换机s2发送Packet-out消息时，可以得知链路源交换机，源接口（s2,port3)。通过收到的Packet-in消息知道目的交换机(s1)。如此，控制器便发现了s1与s2之间的完整链路。
对于存在多个交换机的网络，上述分析过程一样成立。

### example

交换机存储的是MAC地址和端口的对应关系，同一子网主机传输报文通过ip地址，交换机第一次传输的时候不知道IP与MAC的对应关系，因此不知道从哪个端口转发，因此交换机会往所有端口进行转发数据包，而IP和MAC对应的主机就会回复，因此交换机就知道哪个端口对应的主机MAC地址，把这个关系加入到表中，这个过程就是交换机的MAC地址自学习。

一般监控ARP数据包学习MAC地址，将ARP数据包通过Packet-in消息发送给OpenFlow控制器，可以在不过度加重OpenFlow控制器负载的情况下，实现MAC地址学习。

假如PC-a发送数据给PC-b，但是它的ARP表里没有PC-b，所以它需要发送ARP包获取ARP回应以存储到ARP表中。

PC-a发送ARP请求，OpenFlow交换机接收到该请求，检索流表，流表项的匹配字段是入端口，action是上报控制器，可以找找流表项的具体实现代码。

流表的action的输出端口为controller，也就是上报给控制器。

流表项匹配成功后，OpenFlow交换机执行通过Packet-in消息向控制器转发ARP请求帧的action，而控制器要做两个处理，一个是接受Packet-in，二是对Packet-in解码。

为了将来PC-a的ARP请求广播到OpenFlow网络内



ONOS对于Packet-in/out消息的定义

> 我们来看怎么样完成packet-in和packet-out的操作。这两个操作跟OpenFlow是类似的。
>
> 我们看右边，在OpenFlow里面我们也会用到图中的第一块，Packet Service，不同的是在下面，对接的是PI框架的Packet Provider，因为在P4里面，数据包的格式是我们自定义的，所以在这里借助我们的Interpreter进行数据包的解析。解析之后，生成Inbound/Outbound Packet，它们都是ONOS中原有的对packet-in/out的抽象。
>
> 接着，这里借助P4 Runtime设备驱动将其转换成PI Packet Operation，它是PI框架中对包操作的抽象。
>
> 最后，再通过南向插件与设备交互，在这里同样借助P4Info完成PI框架抽象对象与通信报文的转换。
>
> 在ONOS的源码中，现在已经有了一个名为p4-tutorial的示例应用，在onos/apps目录下

![](https://pic.downk.cc/item/5f58a350160a154a6704771b.png)

## Very Simple Switch Architecture

![](https://pic.downk.cc/item/5fa26e441cd1bbb86b19eb8b.jpg)

白色是p4可编程模块，青色是固定模块，红色代表解析的报文流（用户定义的数据流），绿色是数据平面接口（固有元数据暴露的固定功能块和可编程块之间传递信息）

### Arbiter block

固定模块，仲裁器，从ethernet网卡、控制平面、recirculation port这三个port中的一个接受报文。如果从ethernet网卡接受报文，则需要进行校验，从payload去除尾部的校验，如果校验和不对就丢弃。如果报文较多，会触发仲裁算法，如果队列已满，则丢弃数据包。

arbiter设置inctrl.inputport的值，这个值为了表明数据包从哪个端口进入的，是ethernet网卡、控制平面、recirculation port里的哪个。这作为pipeline的输入参数之一，这里用0-7表示网卡端口，13是recirculation端口，14是控制平面端口。

### parser runtime

runtime block与解析器协同工作。如果parser错误的话，它提供error code作为pipeline的输入，并且它提供payload的信息（例如有效载荷的大小）给输出队列demux。parser完成解析数据包，就会使用关联的元数据（packet headers and user-defined metadata）作为输入调用pipeline模块。

### demux

demux的功能是接受deparser构造的headers和parser输出的payload，组装成一个新的数据包并发送到输出端口，输出端口由outctrl.outputport指定，由pipeline设置。指定的端口是四选一：drop、网卡端口（添加校验和）、recirculation给arbiter、控制平面端口（原始数据包，不是deparser构造的数据包，构造的数据包被丢弃）。

### Available extern blocks

校验和功能checksum

### A complete Very Simple Switch program

这是个实现vss架构的P4程序例子，并没有完全调用架构提供的接口，比如recirculation。使用这个例子结合上面介绍的固定模块，实现ipv4数据包的转发。

![](https://pic.downk.cc/item/5fa356fc1cd1bbb86b44659e.jpg)

解析器尝试识别Ethernet头和ipv4头，如果缺少任何一个header，则解析会因错误终止，它将这些headers中的信息放入到parser packet struct中。上图中的四个match-action单元由p4程序中table表示。

第一个表ipv4_match使用ipv4目标地址决定下一跳的outputport和ipv4地址，如果查找失败，则丢弃数据包，该表还减少ttl值。路由表。

第二个表检查ttl值，ttl=0，则通过packet-in发送到控制平面

第三个表使用由第一个表计算的下一跳的ipv4地址来构造目的ethernet地址，也就是通过arp获取ipv4对应的mac地址，arp表

第四个表使用outputport来标识当前交换机的源Ethernet地址，该地址在传出的数据包中设置。

deparser通过重新组装由pipeline计算出的Ethernet和IPv4标头来构造传出数据包。

```c
// Include P4 core library
# include <core.p4>

// Include very simple switch architecture declarations
# include "very_simple_switch_model.p4"

// This program processes packets comprising an Ethernet and an IPv4
// header, and it forwards packets using the destination IP address

typedef bit<48>  EthernetAddress;
typedef bit<32>  IPv4Address;

// Standard Ethernet header
header Ethernet_h {
    EthernetAddress dstAddr;
    EthernetAddress srcAddr;
    bit<16>         etherType;
}

// IPv4 header (without options)
header IPv4_h {
    bit<4>       version;
    bit<4>       ihl;
    bit<8>       diffserv;
    bit<16>      totalLen;
    bit<16>      identification;
    bit<3>       flags;
    bit<13>      fragOffset;
    bit<8>       ttl;
    bit<8>       protocol;
    bit<16>      hdrChecksum;
    IPv4Address  srcAddr;
    IPv4Address  dstAddr;
}

// Structure of parsed headers
struct Parsed_packet {
    Ethernet_h ethernet;
    IPv4_h     ip;
}

// Parser section

// User-defined errors that may be signaled during parsing
error {
    IPv4OptionsNotSupported,
    IPv4IncorrectVersion,
    IPv4ChecksumError
}

parser TopParser(packet_in b, out Parsed_packet p) {
    Checksum16() ck;  // instantiate checksum unit

    state start {
        b.extract(p.ethernet);
        transition select(p.ethernet.etherType) {
            0x0800: parse_ipv4;
            // no default rule: all other packets rejected
        }
    }

    state parse_ipv4 {
        b.extract(p.ip);
        verify(p.ip.version == 4w4, error.IPv4IncorrectVersion);
        verify(p.ip.ihl == 4w5, error.IPv4OptionsNotSupported);
        ck.clear();
        ck.update(p.ip);
        // Verify that packet checksum is zero
        verify(ck.get() == 16w0, error.IPv4ChecksumError);
        transition accept;
    }
}

// Match-action pipeline section

control TopPipe(inout Parsed_packet headers,
                in error parseError, // parser error
                in InControl inCtrl, // input port
                out OutControl outCtrl) {
     IPv4Address nextHop;  // local variable

     /**
      * Indicates that a packet is dropped by setting the
      * output port to the DROP_PORT
      */
      action Drop_action() {
          outCtrl.outputPort = DROP_PORT;
      }

     /**
      * Set the next hop and the output port.
      * Decrements ipv4 ttl field.
      * @param ivp4_dest ipv4 address of next hop
      * @param port output port
      */
      action Set_nhop(IPv4Address ipv4_dest, PortId port) {
          nextHop = ipv4_dest;
          headers.ip.ttl = headers.ip.ttl - 1;
          outCtrl.outputPort = port;
      }

     /**
      * Computes address of next IPv4 hop and output port
      * based on the IPv4 destination of the current packet.
      * Decrements packet IPv4 TTL.
      * @param nextHop IPv4 address of next hop
      */
     table ipv4_match {
         key = { headers.ip.dstAddr: lpm; }  // longest-prefix match
         actions = {
              Drop_action;
              Set_nhop;
         }
         size = 1024;
         default_action = Drop_action;
     }

     /**
      * Send the packet to the CPU port
      */
      action Send_to_cpu() {
          outCtrl.outputPort = CPU_OUT_PORT;
      }

     /**
      * Check packet TTL and send to CPU if expired.
      */
     table check_ttl {
         key = { headers.ip.ttl: exact; }
         actions = { Send_to_cpu; NoAction; }
         const default_action = NoAction; // defined in core.p4
     }

     /**
      * Set the destination MAC address of the packet
      * @param dmac destination MAC address.
      */
      action Set_dmac(EthernetAddress dmac) {
          headers.ethernet.dstAddr = dmac;
      }

     /**
      * Set the destination Ethernet address of the packet
      * based on the next hop IP address.
      * @param nextHop IPv4 address of next hop.
      */
      table dmac {
          key = { nextHop: exact; }
          actions = {
               Drop_action;
               Set_dmac;
          }
          size = 1024;
          default_action = Drop_action;
      }

      /**
       * Set the source MAC address.
       * @param smac: source MAC address to use
       */
       action Set_smac(EthernetAddress smac) {
           headers.ethernet.srcAddr = smac;
       }

      /**
       * Set the source mac address based on the output port.
       */
      table smac {
           key = { outCtrl.outputPort: exact; }
           actions = {
                Drop_action;
                Set_smac;
          }
          size = 16;
          default_action = Drop_action;
      }

      apply {
          if (parseError != error.NoError) {
              Drop_action();  // invoke drop directly
              return;
          }

          ipv4_match.apply(); // Match result will go into nextHop
          if (outCtrl.outputPort == DROP_PORT) return;

          check_ttl.apply();
          if (outCtrl.outputPort == CPU_OUT_PORT) return;

          dmac.apply();
          if (outCtrl.outputPort == DROP_PORT) return;

          smac.apply();
    }
}

// deparser section
control TopDeparser(inout Parsed_packet p, packet_out b) {
    Checksum16() ck;
    apply {
        b.emit(p.ethernet);
        if (p.ip.isValid()) {
            ck.clear();              // prepare checksum unit
            p.ip.hdrChecksum = 16w0; // clear checksum
            ck.update(p.ip);         // compute new checksum.
            p.ip.hdrChecksum = ck.get();
        }
        b.emit(p.ip);
    }
}

// Instantiate the top-level VSS package
VSS(TopParser(),
    TopPipe(),
    TopDeparser()) main;
```

### table

![](https://pic.downk.cc/item/5fa3a3551cd1bbb86b554906.jpg)

## v1model架构

[github refer](https://github.com/p4lang/behavioral-model/blob/master/docs/simple_switch.md)

v1model架构是bmv2正在用的架构，还有一个新的PSA(portable switch architecture)架构正在开发中。

### standard metadata

- ingress port：只读，对于新报文，是数据包到达设备的入口端口号；
- packet_length：对于新报文或者recirculation报文的字节长度，对于克隆或者resubmitted，需要保存这个值，否则是0；
- egress_spec：在ingress代码中控制报文的输出端口。p4_14里的drop或者v1model里的mark_to_drop的action是给这个值分配DROP_PORT（511），这样的话如果在ingress程序的末尾被赋予这个值，该数据包则被丢弃，不会存储在数据包buffer里，也不会发送到egress程序；
- egress_port：只读，在egress程序期间决定数据包要被发送的输出端口；
- egress_instance：重命名egress_ride；
- instance_type：p4代码可以读取的值，在ingress代码部分可以分辨这个数据包是从新端口读取的（normal），还是重新提交的（resubmit），还是循环过来的（recirc）。在egress代码中，这个值可以分辨数据包是否由ingress克隆到egress（ingress_clone），还是egress克隆的（egress_clone），还是在ingress代码中设定的多播（replication）复制的结果，也可以不选择任何一个，就是来自ingress的正常报文（normal）；
- parser_error：erro.NoError是正常或者parser_status=0（p4_14）
- check_sum：只读，对verify_checksum进行调用，如果错误则为1。在ingress之前和parser之后的verifychecksum的控件中调用。

### intrinsic metadata

intrinsic metadata相比standard metadata可以提供更多的功能，但是intrinsic metadata不是必须要提供的。

#### intrinsic_metadata header

如果使用p4_16的v1model架构，这些都被定义在standard_metadata_t的struct里。

如果使用P4_14的simple_switch架构

```c++
header_type intrinsic_metadata_t {
    fields {
        ingress_global_timestamp : 48;
        egress_global_timestamp : 48;
        mcast_grp : 16; //multicast feature
        egress_rid : 16; //multicast featue
    }
}
metadata intrinsic_metadata_t intrinsic_metadata;
```

#### queueing_metadata header

如果使用p4_16的v1model架构，这些都被定义在standard_metadata_t的struct里。

如果使用P4_14的simple_switch架构,并且需要在ingress或者egress pipeline访问队列信息才需要定义

```c++
header_type queueing_metadata_t {
    fields {
        enq_timestamp : 48;
        enq_qdepth : 16;
        deq_timedelta : 32;
        deq_qdepth : 16;
        qid : 8;
    }
}
metadata queueing_metadata_t queueing_metadata;
```

### 支持的actions

在ingress或者egress程序的末尾发生的actions

伪代码ingress process pseudocode：

```c++
if (a clone primitive action was called) {
    create clone(s) of the packet with details configured for the clone session
}
if (digest to generate) {   // because your code called generate_digest
    send a digest message to the control plane software
}
if (resubmit was called) {
    start ingress processing over again for the original packet
} else if (mcast_grp != 0) {  // because your code assigned a value to mcast_grp
    multicast the packet to the output port(s) configured for group mcast_grp
} else if (egress_spec == DROP_PORT) {  // e.g. because your code called drop/mark_to_drop
    Drop packet.
} else {
    unicast the packet to the port equal to egress_spec
}
```

伪代码egress process pseudocode：

```c++
if (a clone primitive action was called) {
    create clone(s) of the packet with details configured for the clone session
}
if (egress_spec == DROP_PORT) {  // e.g. because your code called drop/mark_to_drop
    Drop packet.
} else if (recirculate was called) {
    after deparsing, deparsed packet starts over again as input to parser
} else {
    Send the packet to the port in egress_port.
}
```

### 支持的table match

- `exact` - from P4_16 language specification
- `lpm` - from P4_16 language specification
- `ternary` - from P4_16 language specification
- `optional` - defined in `v1model.p4`
- `range` - defined in `v1model.p4`
- `selector` - defined in `v1model.p4`

如果一个表具有多个lpm键字段，则p4c BMv2后端将拒绝该表；selector只支持具有实现selector action的table。

#### 通过const entries为table match指定匹配条件

example:

```c++
header h1_t {
    bit<8> f1;
    bit<8> f2;
}

struct headers_t {
    h1_t h1;
}

// ... later ...

control ingress(inout headers_t hdr,
                inout metadata_t m,
                inout standard_metadata_t stdmeta)
{
    action a(bit<9> x) { stdmeta.egress_spec = x; }
    table t1 {
        key = { hdr.h1.f1 : range; }
        actions = { a; }
        const entries = {
             1 ..  8 : a(1);
             6 .. 12 : a(2);  // ranges are allowed to overlap between entries
            15 .. 15 : a(3);
            17       : a(4);  // equivalent to 17 .. 17
            // It is not required to have a "match anything" rule in a table,
            // but it is allowed (except for exact match fields), and several of
            // these examples have one.
            _        : a(5);
        }
    }
    table t2 {
        key = { hdr.h1.f1 : ternary; }
        actions = { a; }
        // There is no requirement to specify ternary match criteria using
        // hexadecimal values.  I personally prefer it to make the mask bit
        // positions more obvious.
        const entries = {
            0x04 &&& 0xfc : a(1);
            0x40 &&& 0x72 : a(2);
            0x50 &&& 0xff : a(3);
            0xfe          : a(4);  // equivalent to 0xfe &&& 0xff
            _             : a(5);
        }
    }
    table t3 {
        key = {
            hdr.h1.f1 : optional;
            hdr.h1.f2 : optional;
        }
        actions = { a; }
        const entries = {
            // Note that when there are two or more fields in the key of a
            // table, const entries key select expressions must be surrounded by
            // parentheses.
            (47, 72) : a(1);
            ( _, 72) : a(2);
            ( _, 75) : a(3);
            (49,  _) : a(4);
            _        : a(5);
        }
    }
    table t4 {
        key = { hdr.h1.f1 : lpm; }
        actions = { a; }
        const entries = {
            0x04 &&& 0xfc : a(1);
            0x40 &&& 0xf8 : a(2);
            0x04 &&& 0xff : a(3);
            0xf9          : a(4);  // equivalent to 0xf9 &&& 0xff
            _             : a(5);
        }
    }
    table t5 {
        key = { hdr.h1.f1 : exact; }
        actions = { a; }
        const entries = {
            0x04 : a(1);
            0x40 : a(2);
            0x05 : a(3);
            0xf9 : a(4);
        }
    }
    // ... more code here ...
}
```

## 测试说明

### 测试要求

**Ø** ***\*测试集合2\****·

1. 启动交换机，配置p4程序，default action默认为packet_in, 加载空流表；

2. 打入IPv4数据包，在外部接口处可检测到对应的packet_in数据包；

3. 打入Ipv6数据包，在外部接口处可检测到对应的packet_in数据包；

4. 打入Geo数据包，在外部接口处可检测到对应的packet_in数据包；

**Ø** ***\*测试集合\*******\*3\****

1. 启动交换机，配置p4程序，default action默认为packet_in, 加载空流表；

2. 控制器打入IPv4的packet_out消息，从对应的平台数据接口检测到输出的IPv4数据包；

3. 控制器打入IPv6的packet_out消息，从对应的平台数据接口检测到输出的IPv6数据包；

4. 控制器打入Geo的packet_out消息，从对应的平台数据接口检测到输出的Geo数据包；

测试Packet-In的话，需要把handleRPC函数里的new StreamChannelImpl注释掉，因为这个创建的是server端读模式。

测试packet-out，则需要把StreamChannelImpl类里的new_responder_created(true)改成new_responder_created(false)。