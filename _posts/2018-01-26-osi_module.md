---
layout:     post
title:      "OSI七层模型"
category : basictheory
date:       2018-01-26
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - 
---

# 1. 概述

开放系统互连模型（Open Systems Interconnection model，OSI 模型）是国际标准化组织制定的一种抽象概念，用以在多种通信系统及标准协议间界定与标准化其通信功能与交互操作。原始版本的 OSI 模型分为七层，每一层实现各自的功能和协议，并完成与相邻层的接口通信。OSI 的服务定义详细说明了各层所提供的服务。某一层的服务就是该层及其下各层的一种能力，它通过接口提供给更高一层。各层所提供的服务与这些服务是怎么实现的无关。

![OSI](/img/in-post/osi/tcpip.gif)

# 2. 详述

## 2.1 物理层 Physical Layer

物理层定义了数据连接过程中的电气及物理规格（如设备引脚、电压、线路阻抗等），以及设备与物理传输介质（如电缆、光纤电缆或射频链路）之间的关系。物理层负责在物理介质中接收和传输非结构化数据，也即是比特流，它不关心协议或其它这样的高级项目，是整个 OSI 模型的基础。比特流控制是在物理层完成的。

光纤、同轴电缆、双绞线、中继器、集线器等都是物理层的常见设备。

## 2.2 数据链路层 Data Link Layer

数据链路层提供直连节点直接的数据传输，检测并纠正物理层中可能出现的错误。在物理层提供比特流服务的基础上，将比特信息封装成数据帧 Frame，起到在物理层上建立、撤销、标识逻辑链接和链路复用以及差错校验等功能。数据链路层建立相邻结点之间的数据链路，通过差错控制提供数据帧在信道上无差错的传输，同时为其上面的网络层提供有效的服务。

IEEE 802 系列标准把数据链路层分成 LLC（Logical Link Control，逻辑链路控制）和 MAC（Media Access Control，介质访问控制）两个子层。上面的 LLC 子层实现数据链路层与硬件无关的功能，负责识别和封装网络层协议，并控制错误检查和帧同步；较低的 MAC 子层提供 LLC 和物理层之间的接口，负责控制网络中的设备如何访问介质并允许传输数据。

数据链路层的典型设备有二层交换机、网桥、网卡等。

## 2.3 网络层 Network Layer

网络层通过 IP 寻址来建立两个节点之间的连接，提供将变长数据序列（称作数据报）在节点间传输的功能和手段。网络层选择合适的网间路由和交换结点，正确无误地按照地址传送给目的端。除个别网络层协议，网络层本身不保证传输的可靠性，也不必报告传输错误。

如果消息太大而无法在数据链路层上从一个节点传送到另一个节点，则网络层可以在一个节点处将消息分割成若干片段独立发送，在另一个节点重新组合。

网络层的典型设备有三层交换机、网关、路由器等。

## 2.4 传输层 Transport Layer

传输层为上层协议提供端到端的可靠和透明的数据传输服务。所谓可靠，依赖的是流量控制、分片重组、差错控制机制；所谓透明，是指该层向高层屏蔽了下层数据通信的细节，使高层用户看到的只是在两个传输实体间的一条主机到主机的、可由用户控制和设定的、可靠的数据通路。

OSI 定义了五类连接模式的传输协议，从提供最少的功能的 TP0 到 设计用于不太可靠网络（如 Internet）的 TP4，详细特性如下表：

Feature | TP0 | TP1 | TP2 | TP3 | TP4
--- | --- | --- | --- | --- | ---
Connection-oriented network | Yes | Yes | Yes | Yes | Yes
Connectionless network | No | No | No | No | Yes
Concatenation and separation | No | Yes | Yes | Yes | Yes
Segmentation and reassembly | Yes | Yes | Yes | Yes | Yes
Error recovery | No | Yes | Yes | Yes | Yes
Reinitiate connectiona | No | Yes | No | Yes | No
Multiplexing / demultiplexing over single virtual circuit | No | No | Yes | Yes | Yes
Explicit flow control | No | No | Yes | Yes | Yes
Retransmission on timeout | No | No | No | No | Yes
Reliable transport service | No | Yes | No | Yes | Yes

## 2.5 会话层 Session Layer

会话层负责建立、管理和终止表示层实体之间的通信会话。该层的通信由不同设备中的应用程序之间的服务请求和响应组成。

OSI 模型使用该层来优雅地关闭会话，这也是 TCP 协议的属性之一。在非 IP 协议集中，该层还提供会话检查点（session checkpointing）与恢复（recovery）功能。

## 2.6 表示层 Presentation Layer

表示层提供各种用于应用层数据的编码和转换功能,确保一个系统的应用层发送的数据能被另一个系统的应用层识别。如果必要，该层可提供一种标准表示形式，用于将计算机内部的多种数据格式转换成通信中采用的标准表示形式。数据压缩和加密也是表示层可提供的转换功能之一。

## 2.7 应用层 Application Layer

OSI参考模型中最靠近用户的一层，是为计算机用户提供应用接口，也为用户直接提供各种网络服务。应用层功能通常包括识别通信伙伴，确定资源可用性和同步通信。

应用实体与应用的区别，是理解应用层的一个重要概念。例如，一个订票网站有两个应用实体，一个使用 HTTP 协议与其客户交互，一个使用远程数据库协议记录预定信息，但这两个协议都和订票业务没有关系，这只是程序的内部逻辑。应用层本身无法确定网络中资源的可用性。

# 3. 参考

1. [维基百科](https://en.wikipedia.org/wiki/OSI_model)