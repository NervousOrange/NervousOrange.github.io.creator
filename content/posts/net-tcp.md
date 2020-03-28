---
title: "Net：运输层"
author: "Chenghao Zheng"
tags: ["Net"]
categories: ["Study notes"]
date: 2020-03-12T13:19:47+01:00
draft: false
---


本篇为[Net：概述与应用层](https://chenghao.monster/2020/net-http/) 续篇，[《计算机网络自顶向下方法》](https://book.douban.com/subject/30280001/) 第三的读书笔记，主要介绍：TCP 协议、流量控制、UDP 协议 等。

# 运输层

### TCP 报文段结构

下图显示了 TCP 报文段的结构，其首部包括源端口号和目的端口号，它被用于多路复用/分解来自或送到上层应用的数据。

*  检验和字段（checksum field）
* 32 Bits 的序号字段和 32 Bits 的确认号字段，用来实现可靠数据传输服务
* 16 Bits 的接收窗口字段，用于流量控制
* 6 Bits 的 **标志字段**，ACK Bit 用于指示确认字段中的值是有效的，即该报文段包括一个对已被成功接收报文段的确认 。 RST、SYN 和 FIN 比特用于连接建立和拆除，  

![](/images/TCP报文结构.png)

### 可靠数据传输

TCP 在 lP 不可靠的尽力而为服务之上创建了 一种 **可靠数据传输服务**，确保一个进程从其接收缓存中读出的数据流是无损坏、无间隔、非冗余和按序的数据流；即该字节流与连接的另 一方端系统发送出的字节流是完全相同 。   （丢包重传、超时重传）

通过 **三次握手** 来建立一条 TCP 连接：

1. 第一次握手：客户端通过向服务器发送一段含有同步标志（SYN = 1）的数据，向服务器请求建立连接，通过这个数据段，客户端告诉服务器，**我想要和你通信**，你可以使用哪个序列号作为起始数据段来回应我。
2. 第二次握手：服务器收到客户端的请求后，为该 TCP 连接分配 TCP 缓存和变量，用一个带有确认应答（ACK）和同步序列号（SYN = 1）标志位的数据段响应客户端，也告诉服务器两件事情：**1.我已经收到你的请求，你可以传输数据了。2.你要用那个数据段来回应我**。
3. 第三次握手：客户端收到这个数据段之后，也要给该连接分配缓存和变量，再发送一个确认应答，确认已经收到服务器的数据段。告诉服务器，**我已经收到回复，现在可以传输实际数据了**。

![](/images/三次握手.png)

通过 **四次挥手** 来断开一条 TCP 连接：

1.  第一次，当客户端完成数据传输后，将控制位 FIN 置 1，**提出停止 TCP 连接的请求**。
2. 第二次，服务器收到 FIN 后对其作出确认响应，确认这一方向上的 TCP 连接将关闭，将 ACK 置1。
3. 第三次，由服务器再 **提出反方向的关闭请求**，将 FIN 置 1。
4. 第四次，客户端对服务器的请求进行确认，将 ACK 置 1，双方关闭结束，释放用于该连接的所有资源。
5.  为什么需要四次挥手：客户端在数据传输结束后发出连续释放的通知，待对方确认后进入半关闭状态；服务器在传输完最后一段数据后，也发出释放通知，待对方确认后再完全关闭 TCP 连接。

![](/images/四次挥手.png)

### 流量控制

每一侧主机都为 TCP 连接设置了接收缓存，接收到的数据会被放入接收缓存，相关的应用进程会从该缓存中读取数据。如果某应用程序 **读取数据时相对缓慢**，而发送方发送得太多 、 太快，发送的数据就会很容易地使该连接的接收缓存溢出 。  

TCP 为其应用程序提供了流量控制服务，以 **消除发送方使接收方缓存溢出的可能性**。流量控制因此是个速度匹配服务。TCP 是全双工通信，在连接两端的发送方都各自维护一个 `接收窗口`，给发送方一个指示 —— 该接收方还有多少可用的缓存空间，以实现流量控制。

![](/images/流量控制.png)

与之相对的，TCP 发送方也可能因为 IP 网络的拥塞而被遏制（通过拥塞窗口），称为 **拥塞控制**。

### UDP 协议

使用 UDP 时，因为发送方和接收方的运输层实体之间没有握手，UDP 也被称为是无连接的。 UDP 相比 TCP 有以下几个特点：

* 没有流量控制和拥塞控制，更适合实时应用。
* 无需握手，不会引入建立连接的时延。
* UDP 不需要维护连接状态，一般能支持更多的活跃客户。
* 相比 TCP 20 个字节的首部开销，UDP 仅有 8 字节。
