---
layout: post
title: 网络相关概念总结
category: 网络
---
---

## 一 概述

### 1.1 计算机网络体系结构：

* OSI：7层。从上往下：应用层、表示层、会话层、运输层、网络层、数据链路层、物理层
* 五层协议：5层。应用层、运输层、网络层、数据链路层、物理层。
* TCP/IP:4层。应用层、运输层、网际层、网络接口层。

  数据在各层中传递：在向下的过程中，需要添加下层协议所需要的首部或者尾部，而在向上的过程中不断拆开首部和尾部。路由器只有下面三层协议，因为路由器位于网络核心中，不需要为进程或者应用程序提供服务，因此也就不需要传输层和应用层。

## 二 物理层

### 2.1 通信方式

根据信息在传输线上的传送方向，分为以下三种通信方式：

- **单工通信：**单向传输
- **半双工通信：**双向交替传输
- **全双工通信：**双向同时传输

## 三 链路层

### 3.1 基本概念

#### 3.1.1  封装成帧

　　将网络层传下来的数据包添加首部和尾部，用于标记帧的开始和结束。

#### 3.1.2 透明传输

　　透明表示一个实际存在的事物看起来好像不存在一样。帧使用首部和尾部进行定界，如果帧的数据部分含有和首部尾部相同的内容，那么帧的开始和结束位置就会被错误的判定。需要在数据部分出现首部尾部相同的内容前面插入转义字符。如果数据部分出现转义字符，那么就在转义字符前面再加个转义字符。在接收端进行处理之后可以还原出原始数据。这个过程透明传输的内容是转义字符，用户察觉不到转义字符的存在。

#### 3.1.3 差错检测

　　目前数据链路层广泛使用了循环冗余检验（CRC）来检查比特差错。

### 3.2 信道分类

#### 3.2.1 广播信道

　　一对多通信，一个节点发送的数据能够被广播信道上所有的节点接收到。所有的节点都在同一个广播信道上发送数据，因此需要有专门的控制方法进行协调，避免发生冲突（冲突也叫碰撞）。主要有两种控制方法进行协调，一个是使用信道复用技术，一是使用 CSMA/CD 协议。

#### 3.2.2 点对点信道

　　一对一通信。因为不会发生碰撞，因此也比较简单，使用 PPP 协议进行控制。

### 3.3 信道复用

#### 3.3.1. 频分复用

　　频分复用的所有主机在相同的时间占用不同的频率带宽资源。

#### 3.3.2 时分复用

　　时分复用的所有主机在不同的时间占用相同的频率带宽资源。使用频分复用和时分复用进行通信，在通信的过程中主机会一直占用一部分信道资源。但是由于计算机数据的突发性质，通信过程没必要一直占用信道资源而不让出给其它用户使用，因此这两种方式对信道的利用率都不高。

#### 3.3.3 统计时分复用

　　是对时分复用的一种改进，不固定每个用户在时分复用帧中的位置，只要有数据就集中起来组成统计时分复用帧然后发送。

#### 3.3.4 波分复用

　　光的频分复用。由于光的频率很高，因此习惯上用波长而不是频率来表示所使用的光载波。

#### 3.3.5 码分复用

### 3.4 相关协议

#### 3.4.1 CSMA/CD 协议

　　CSMA/CD 表示载波监听多点接入 / 碰撞检测。

- 多点接入 ：说明这是总线型网络，许多主机以多点的方式连接到总线上。
- 载波监听 ：每个主机都必须不停地监听信道。在发送前，如果监听到信道正在使用，就必须等待。
- 碰撞检测 ：在发送中，如果监听到信道已有其它主机正在发送数据，就表示发生了碰撞。虽然每个主机在发送数据之前都已经监听到信道为空闲，但是由于电磁波的传播时延的存在，还是有可能会发生碰撞。

记端到端的传播时延为 τ，最先发送的站点最多经过 2τ 就可以知道是否发生了碰撞，称 2τ 为 争用期 。只有经过争用期之后还没有检测到碰撞，才能肯定这次发送不会发生碰撞。

当发生碰撞时，站点要停止发送，等待一段时间再发送。这个时间采用 截断二进制指数退避算法 来确定。从离散的整数集合 {0, 1, .., (2k-1)} 中随机取出一个数，记作 r，然后取 r 倍的争用期作为重传等待时间。

#### 3.4.2 PPP协议

互联网用户通常需要连接到某个 ISP 之后才能接入到互联网，PPP 协议是用户计算机和 ISP 进行通信时所使用的数据链路层协议。 

### 3.5 MAC地址

MAC 地址是链路层地址，长度为 6 字节（48 位），用于唯一标识网络适配器（网卡）。一台主机拥有多少个网络适配器就有多少个 MAC 地址。例如笔记本电脑普遍存在无线网络适配器和有线网络适配器，因此就有两个 MAC 地址。

### 3.6 局域网

局域网是一种典型的广播信道，主要特点是网络为一个单位所拥有，且地理范围和站点数目均有限。主要有以太网、令牌环网、FDDI 和 ATM 等局域网技术，目前以太网占领着有线局域网市场。

#### 3.6.1 以太网

以太网是一种星型拓扑结构局域网。

早期使用集线器进行连接，集线器是一种物理层设备， 作用于比特而不是帧，当一个比特到达接口时，集线器重新生成这个比特，并将其能量强度放大，从而扩大网络的传输距离，之后再将这个比特发送到其它所有接口。如果集线器同时收到两个不同接口的帧，那么就发生了碰撞。

目前以太网使用交换机替代了集线器，交换机是一种链路层设备，它不会发生碰撞，能根据 MAC 地址进行存储转发。

以太网帧格式：

- **类型** ：标记上层使用的协议；

- **数据** ：长度在 46-1500 之间，如果太小则需要填充；

- **FCS** ：帧检验序列，使用的是 CRC 检验方法

#### 3.6.2 交换机

　　交换机具有自学习能力，学习的是交换表的内容，交换表中存储着 MAC 地址到接口的映射。正是由于这种自学习能力，因此交换机是一种即插即用设备，不需要网络管理员手动配置交换表内容。

## 四 网络层

### 4.1 IP协议

>  **在相互连接的网络中传递IP数据报。** 

#### 4.1.1 作用：

**寻址与路由：**

- **寻址：**用IP地址唯一标识主机，在数据报中，携带源主机IP地址与目的主机IP地址。
- **路由：**在数据报的传输中，每个中间节点（网关）需要选择从源主机到目的主机的合适转发路径。

**分段与重组:**

- **分段：**IP数据报受不同类型的通信网络所规定的最大传输单元MTU的限制而将数据报分段；
- **重组：**将IP数据报分割成一个个小数据包后他们独立地在网络中转发，到大目的主机后被重组，恢复成原来的数据报。

因为网络层是整个互联网的核心，因此应当让网络层尽可能简单。网络层向上只提供简单灵活的、无连接的、尽最大努力交互的数据报服务。使用 IP 协议，可以把异构的物理网络连接起来，使得在网络层看起来好像是一个统一的网络。

与 IP 协议配套使用的还有三个协议：

- 地址解析协议 ARP（Address Resolution Protocol）
- 网际控制报文协议 ICMP（Internet Control Message Protocol）
- 网际组管理协议 IGMP（Internet Group Management Protocol）

#### 4.1.2 IP 数据报格式

- 版本 : 有 4（IPv4）和 6（IPv6）两个值；
- 首部长度 : 占 4 位，因此最大值为 15。值为 1 表示的是 1 个 32 位字的长度，也就是 4 字节。因为首部固定长度为 20 字节，因此该值最小为 5。如果可选字段的长度不是 4 字节的整数倍，就用尾部的填充部分来填充。
- 区分服务 : 用来获得更好的服务，一般情况下不使用。
- 总长度 : 包括首部长度和数据部分长度。
- 生存时间 ：TTL，它的存在是为了防止无法交付的数据报在互联网中不断兜圈子。以路由器跳数为单位，当 TTL 为 0 时就丢弃数据报。
- 协议 ：指出携带的数据应该上交给哪个协议进行处理，例如 ICMP、TCP、UDP 等。
- 首部检验和 ：因为数据报每经过一个路由器，都要重新计算检验和，因此检验和不包含数据部分可以减少计算的工作量。
- 标识 : 在数据报长度过长从而发生分片的情况下，相同数据报的不同分片具有相同的标识符。
- 片偏移 : 和标识符一起，用于发生分片的情况。片偏移的单位为 8 字节。

#### 4.1.3 IP地址分类

1. 子网掩码划分

网络地址下的的主机通信叫同一网段下的通信，此种通信将数据直接发送；在不同网络地址下的主机通信叫不同网段下的主机通信，需要将数据交给网关，由网关转发。

由子网掩码确定是否两个主机处于同一网段，用法：将IP地址与子网掩码作逻辑与运算，就得到网络地址，进而就可以判断。

2. ip/前缀表示

无分类编址 CIDR 消除了传统 A 类、B 类和 C 类地址以及划分子网的概念，使用网络前缀和主机号来对 IP 地址进行编码，网络前缀的长度可以根据需要变化。 

IP 地址 ::= {< 网络前缀号 >, < 主机号 >}

CIDR 的记法上采用在 IP 地址后面加上网络前缀长度的方法，例如 128.14.35.7/20 表示前 20 位为网络前缀。
CIDR 的地址掩码可以继续称为子网掩码，子网掩码首 1 长度为网络前缀的长度。
一个 CIDR 地址块中有很多地址，一个 CIDR 表示的网络就可以表示原来的很多个网络，并且在路由表中只需要一个路由就可以代替原来的多个路由，减少了路由表项的数量。把这种通过使用网络前缀来减少路由表项的方式称为路由聚合，也称为 构成超网 。
在路由表中的项目由“网络前缀”和“下一跳地址”组成，在查找时可能会得到不止一个匹配结果，应当采用最长前缀匹配来确定应该匹配哪一个。

### 4.2 地址解析协议 ARP

**网络层实现主机之间的通信，而链路层实现具体每段链路之间的通信。因此在通信过程中，IP 数据报的源地址和目的地址始终不变，而 MAC 地址随着链路的改变而改变。**

**IP地址是网络中的逻辑位置，可变的；而MAC地址是物理物质，不可变。**

**ARP 实现由 IP 地址得到 MAC 地址。**

每个主机都有一个 ARP 高速缓存，里面有本局域网上的各主机和路由器的 IP 地址到 MAC 地址的映射表。

如果主机 A 知道主机 B 的 IP 地址，但是 ARP 高速缓存中没有该 IP 地址到 MAC 地址的映射，此时主机 A 通过广播的方式发送 ARP 请求分组，主机 B 收到该请求后会发送 ARP 响应分组给主机 A 告知其 MAC 地址，随后主机 A 向其高速缓存中写入主机 B 的 IP 地址到 MAC 地址的映射。

### 4.3  网际控制报文协议 ICMP

> **ICMP 是为了更有效地转发 IP 数据报和提高交付成功的机会。**它封装在 IP 数据报中，但是不属于高层协议。
>
> ICMP 报文分为差错报告报文和询问报文。

#### 4.3.1 Ping

Ping 是 ICMP 的一个重要应用，主要用来测试两台主机之间的连通性。

Ping 的原理是通过向目的主机发送 ICMP Echo 请求报文，目的主机收到之后会发送 Echo 回答报文。Ping 会根据时间和成功响应的次数估算出数据包往返时间以及丢包率。

#### 4.3.2 Traceroute

Traceroute 是 ICMP 的另一个应用，用来跟踪一个分组从源点到终点的路径。

Traceroute 发送的 IP 数据报封装的是无法交付的 UDP 用户数据报，并由目的主机发送终点不可达差错报告报文。

### 4.4 网络地址转换 NAT

**专用网内部的主机使用本地 IP 地址又想和互联网上的主机通信时，可以使用 NAT 来将本地 IP 转换为全球 IP。否则数据传输会失败。**

在以前，NAT 将本地 IP 和全球 IP 一一对应，这种方式下拥有 n 个全球 IP 地址的专用网内最多只可以同时有 n 台主机接入互联网。为了更有效地利用全球 IP 地址，现在常用的 NAT 转换表把传输层的端口号也用上了，使得多个专用网内部的主机共用一个全球 IP 地址。使用端口号的 NAT 也叫做网络地址与端口转换 NAPT。

#### 4.4.1 路由器

- 概念：路由器是连接[因特网](https://baike.baidu.com/item/%E5%9B%A0%E7%89%B9%E7%BD%91/114119)中各[局域网](https://baike.baidu.com/item/%E5%B1%80%E5%9F%9F%E7%BD%91/98626)、[广域网](https://baike.baidu.com/item/%E5%B9%BF%E5%9F%9F%E7%BD%91/422004)的设备，它会根据信道的情况自动选择和设定路由，以最佳路径，按前后顺序发送信号。 

- 工作原理：路由器的某个输入端口会受到分组，会根据分组的目的地址，将分组从合适的输出端口发送给下一跳路由器。下一跳路由器也实现同样的功能，直至把分组送到目的地址。

- 功能划分：路由选择和分组转发。 

- 组成结构：交换结构、一组输入端口和一组输出端口 

- 路由学则协议：路由选择协议都是自适应的，能随着网络通信量和拓扑结构的变化而自适应地进行调整。
1. 内部网关协议   RIP
2. 内部网关协议   OSPF
3. 外部网关协议   BGP
