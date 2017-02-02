---
layout: post
title: IP Protocol
category : protocol-model
author: Max
tags : [ip, protocol]
---


# **RFC**

IP协议文档：[RFC 791](http://datatracker.ietf.org/doc/rfc791/?include_text=1)

***

# **协议层次结构**

可以看出IP协议在互联网协议层次机构中德所处位置：


                 +------+ +-----+ +-----+     +-----+  
                 |Telnet| | FTP | | TFTP| ... | ... |  
                 +------+ +-----+ +-----+     +-----+  
                       |   |         |           |     
                      +-----+     +-----+     +-----+  
                      | TCP |     | UDP | ... | ... |  
                      +-----+     +-----+     +-----+  
                         |           |           |     
                      +--------------------------+----+
                      |    Internet Protocol & ICMP   |
                      +--------------------------+----+
                                     |                 
                        +---------------------------+  
                        |   Local Network Protocol  |  
                        +---------------------------+  

                         Protocol Relationships


***

# **IP头**

## 头结构


        0                   1                   2                   3   
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |Version|  IHL  |Type of Service|          Total Length         |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |         Identification        |Flags|      Fragment Offset    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |  Time to Live |    Protocol   |         Header Checksum       |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                       Source Address                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Destination Address                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Options                    |    Padding    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                      Example Internet Datagram Header


  1. Version ： 4 bits

    本文介绍版本4。

  2. Internet Header Length ： 4 bits

    是32位模式下的长度。正确的最小长度为5（5表示头部长度值为5*4=20 Octets)。

  3. Type of Service :  1 byte

          Bits 0-2:  Precedence(优先级). 
          Bit    3:  0 = Normal Delay(正常延迟),        1 = Low Delay(低延迟). 
          Bits   4:  0 = Normal Throughput(正常吞吐),   1 = High Throughput(高吞吐).      
          Bits   5:  0 = Normal Relibility(正常可靠性), 1 = High Relibility(高可靠性).       
          Bit  6-7:  Reserved for Future Use(保留).


             0     1     2     3     4     5     6     7
          +-----+-----+-----+-----+-----+-----+-----+-----+
          |                 |     |     |     |     |     |
          |   PRECEDENCE    |  D  |  T  |  R  |  0  |  0  |
          |                 |     |     |     |     |     |
          +-----+-----+-----+-----+-----+-----+-----+-----+

             Precedence

              111 - Network Control
              110 - Internetwork Control
              101 - CRITIC/ECP
              100 - Flash Override
              011 - Flash
              010 - Immediate
              001 - Priority
              000 - Routine

  4. Total Length :  2 bytes

    数据报的总长度，以字节（octets）为单位计量，包含internet头部和数据。

    最大的internet头部是60个字节，通常internet头部是20个字节，允许高层协议流出一个头部富余量。

  5. Identification :  2 bytes

    该标识由发送者设定，有助于重组数据报的分片。

  6. Flags :  3 bits

          Bit 0: 保留，必须为0        
          Bit 1: (DF) 0 = 可以分片(May Fragment),     1 = 不可分片( Don't Fragment)       
          Bit 2: (MF) 0 = 最后一个分片(Last Fragment)，1 = 还有分片(More Fragments)


              0   1   2
            +---+---+---+
            |   | D | M |
            | 0 | F | F |
            +---+---+---+

  7. Fragment Offset :  13 bits

    该头部指示了这个分片在所属数据报中的位置。分片偏移以8字节(64位)作为计量单位。第一个分片的偏移为0。

  8. Time to Live :  1 byte

    数据报允许在internet系统中生存的最大时间。

  9. Protocol :  1 byte

    下一个层次的协议ID。

  10. Header Checksum :  2 bytes

    仅用于头部。

  11. Source Address :  4 bytes

  12. Destination Address :  4 bytes

  13. Options :  variable

    选项可以出现也可以不出现在数据报中。所有的IP模块(主机和网关)必须实现这个东东。可选的是在任一个特殊数据报中它们的传输，而不是它们的实现。
