---
layout: post
title: GPRS Tunnelling Protocol
category : 协议与模型
tagline:
tags : [gtp, protocol]
---
{% include JB/setup %}

# **协议概述**

   GTP协议（GPRS Tunnelling Protocol）应用在GPRS 骨干网中的GSNs 之间（如 SGSN 和 GGSN 之间），为各个移动台(MS) 建立GTP 通道。GTP 通道是 GPRS服务节点(GSN) 之间的安全通道，两个主机可通过该通道交换数据。

　　SGSN 从 MS 接收数据包，并在 GTP 包头中对其进行封装，然后才通过GTP 通道将其转发到 GGSN。GGSN 接收这些数据包时，先将它们解封，然后转发到外部主机。

　　GTP 数据包包头含有Sequence Number字段，Sequence Number向GGSN(接收 GTP 数据包的GGSN) 指示数据包的顺序。

　　在PDP 环境激活阶段，发送端GGSN 向接收端GGSN 发送的第一个G-PDU 的序列号值是零 (0) ，发送端 GGSN 为其随后发送的每个 G-PDU 增加序列号值，G-PDU序列号值达到 65535 时，重置为零。

　　一般情况下，接收端 GGSN 会校验所接收的数据包中的序列号，接受端GGSN拿自身的计数器序列号和所接收的数据包中的序列号进行比较，如果这两个序号匹配上了，则 GGSN 转发该数据包，如果它们不匹配，则GGSN 丢弃该数据包。


***
# **头结构**


       0                             1               2               3
       0  1  2      3        4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6    7
      +-+---+-+-------------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+
      |Version|Protocol Type| flags | Message Type  |          Length                     |
      +-+---+-+-------------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+
      |                             TEID                                                  |
      +-+---+-+-------------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+
      |           Sequence number                   |  N-PDU number |next extension header|
      +-+---+-+-------------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-------+

                            Example GTP Header


  1. Version : 3 bits

    版本号，目前基本都是1，即GTPv1版本。

  2. Protocol type : 1 bit

        1 为  GTP'   --   GPRS计费采集协议
        0 为  GTP-C  --   信令控制协议
             或GTP-U --   封装用户数据协议

  3. flags : 4 bits

        bit 0 : Reserved
        bit 1 : is next extension header present ?
        bit 2 : is sequence number present ?
        bit 3 : is N-PDU number present?

    后三个bit位，只要有一个置位非0，则这三项都同时存在。

  4. Message Type : 1 byte

        0xff   --    T-PDU

  5. Length : 2 bytes

    GTP封装数据的长度，不包括GTP头部的基本长度，但包括gtp选项的长度，即上面的next extension header，sequence number项和n-pdu number项的长度是包括在这个Total length之中的。

  6. TEID : 4 bytes

    具体是3GPP TS 23.060和3GPP TS 29.060 ？

    Tunnel Endpoint Identifier，GTPv1的概念，用于表示一条隧道(PDP)，分为数据面TEID(TEID(U))和控制面TEID(TEID(C))，由SGSN和GGSN自己分配。

  7. Sequence number : 2 bytes

    对于信令消息而言是交互的标识，对于T-PDU而言是顺序号。

  8. N-PDU number : 1 byte

    用于越SGSN切换时协调MS与SGSN之间的数据传送。

  9. next extension header : 1 byte

    如果next extension header不为0，则下面还有extension header，直到extension header的next extension header项为0。

    Extension header的格式为 `| total length | content | next extension header |` 。其中，total length 和 next extension header 的长度为 1 字节，content 长度不定， 但必须保证 Extension header 的总长度为 4 字节的整数倍。


***

# **信令平面**

## 概述

   承载GTP的协议（UDP或TCP）称为路径协议（Path Protocol），每个GSN对之间有多个路径，每个路径上可以存在多个隧道，GTP负责隧道的建立、使用、管理和释放。信令平面主要包含路径管理、隧道管理、位置管理、移动性管理四大类。

   对于信令消息，GTP包头中的字段设置如下：

  * SNN置为0；
  * SNDCP N-PDULLC Number不使用，发端将该字段置为255，收端忽略该字段；
  * 对于一个路径或一个隧道上的信令，其request类消息的Sequence Number是唯一的，相应的response消息的Sequence Number从接收的request消息的该字段拷贝；
  * 对于路径管理消息、位置管理消息、移动性管理消息而言，TID置为0；对于隧道管理消息而言，TID用于指出目的端GSN的MM和PDP上下文；
  * 对于路径管理消息、位置管理消息而言，Flow Lable不使用而置为0；对于移动性管理消息、隧道管理消息而言，Flow Lable应该指出其试图操作的GTP流，例如在Create PDP Context Request消息中分配GTP流的Flow Lable，在对应的Create PDP Context Response消息中应该包含该字段。

    GPRS信令消息格式与电信界其他信令定义方式类似，每条消息都包含若干个参数（Information Element），参数分为TV（Type，Value）和TLV（Type，Length，Value）两种类型，TV参数的Type字节的最高位为0，TLV参数的Type字节的最高位为1。每个参数都具有唯一的Type标识。

## 路径管理消息

  路径管理是所有其它管理活动的基础，只有确保消息传送的通路处于可用状态，消息才能被正确发送与接收。具体的路径管理消息如下：

  * Echo Request/Response(类型码1/2)消息主要用于探测对端GSN是否仍然处于激活状态，路径是否畅通；
  * Version Not supported(类型码3)用于指示对端版本不符，同时指出本节点支持的版本。

## 隧道管理消息

  隧道管理围绕PDP上下文展开，是GPRS会话过程的核心，具体消息类型如下：

  * Create PDP Context Request/Response(类型码16/17)消息用于PDP上下文的建立请求和响应，SGSN向GGSN发请求消息，GGSN向SGSN发响应消息；
  * Update PDP Context Request/Response(类型码18/19)消息用于PDP上下文的更新请求和响应，当PDP上下文的有关参数如QoS轮廓需要重新协商时，或者手机发生越SGSN切换时，SGSN需要向GGSN发送请求消息，GGSN向SGSN发响应消息；
  * Delete PDP Context Request/Response(类型码20/21)消息用于PDP上下文的删除请求和响应，SGSN或GGSN向对端发请求消息，对端发响应消息；
  * Create AA PDP Context Request/Response(类型码22/23)用于匿名接入情况下建立PDP上下文的请求和响应；
  * Delete AA PDP Context Request/Response(类型码24/25)消息用于匿名接入情况下PDP上下文的删除请求和响应；
  * Error Indication(类型码26)用于错误指示，当SGSN收到G-PDU而相应的PDP上下文不存在或未激活时，当GGSN收到G-PDU而相应的MM上下文不存在或未激活时，SGSN或GGSN向对端发送Error Indication消息；
  * PDU Notification Request/Response(类型码27/28)用于当网络侧发起PDP上下文激活过程时，由GGSN向SGSN发请求消息，SGSN向GGSN发响应消息；
  * PDU Notification Reject Request/Response(类型码29/30)当PDU Notification Request/Response消息交互完成后，但无法激活PDP上下文（如手机拒绝），SGSN向GGSN发PDU Notification Reject Request消息，GGSN发响应消息。

## 位置管理消息

  GGSN定义的相应的GTP域的位置管理消息：

  * Send Routeing Information for GPRS Request/Response(类型码32/33)当网络侧发起PDP上下文激活过程时，GGSN需要MS对应的SGSN地址。GGSN向信令网关GSN发Send Routeing Information for GPRS Request，信令网关GSN会将该消息转换为相应的MAP信令与HLR通信，得到查询结果后发Send Routeing Information for GPRS Response消息给GGSN。
  * Failure Report Request/Response(类型码34/35)GGSN向信令网关GSN发请求消息，用于将MNRG（Mobile station Not Reachable for GPRS flag）置位，信令网关发响应消息。
  * Note MS GPRS Present Request/Response(类型码36/37)GGSN向信令网关GSN发请求消息，通知MS又可以参加通信，信令网关发响应消息。

## 移动性管理消息

  有一部分移动性管理在SGSN之间发生，这些管理消息用GTP协议在承载，这些移动性管理消息如下：

  * Identification Request/Response(类型码48/49)当手机从附着状态转为分离状态，随后移动到新的SGSN区域，而后又转入附着状态，这时新SGSN会向老的SGSN发送Identification Request消息以索取IMSI，老的SGSN发响应消息。
  * SGSN Context Request/Response/Acknowledge(类型码50／51／52)当手机发生越SGSN切换时，新的SGSN会向老的SGSN索取PDP上下文信息。新老SGSN之间会按照该3-way的消息过程进行。


***

# **传输平面**

  传输平面用于传输T-PDU（Tunneled PDU），其路径协议可以采用TCP或UDP，当GTP承载面向连接可靠性要求高的用户协议（如X.25）时，路径可选用TCP；当GTP承载无连接可靠性要求不高的用户协议（如IP）时，路径可选用UDP。

  传输平面中GTP包头有关字段的说明：

  * 如果SNN置为1，那么GTP报头中将包含SNDCP N-PDU Number字段；
  * T-PDU的消息类型是255（十进制）；
  * SNDCP N-PDU Number：当发生SGSN之间切换时，老的SGSN用该字段通知新SGSN关于该T-PDU的N-PDU的序号。如果SNDCP没有分配N-PDU序号，或者采用非确认LLC工作模式，则SNN=0，而N-PDU序号置为255。

  对于传输平面，GTP协议最常用于SGSN与GGSN之间。考虑到SGSN之间会发生切换，老的SGSN可能会残存一部分数据尚未发送给手机；而此时手机已经完成了位置更新，那么这部分残存数据会通过新的SGSN发送到手机，因此SGSN之间也存在GTP传输平面。对于GGSN之间则不存在传输平面，它们之间的数据传送需经过Gi接口，但是这并不意味着GGSN之间必须通过外部数据网作沟通。GGSN之间可以采取直连或通过骨干网相连，只不过数据传送不在GTP协议之上进行而已。
