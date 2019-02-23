---
layout:     post
title:      802.11n帧聚合技术
subtitle:   802.11n帧聚合技术
date:       2019-02-23
author:     naiquan.hu
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- MAC, 11N, 帧聚合
---



# 802.11n帧聚合技术

Author: naiquan.hu
Date: 2019-02-23

---
[TOC]
---


# 1. 802.11n MAC帧格式

为了提升整个网络的吞吐量，IEEE80211系列标准使用了复杂的MAC协议，也使用了相当复杂的MAC帧。80211n标准是对80211标准的改进，其MAC帧通用格式如下：
![](\images\wifi\wifi_0001.png)

# 2. 帧聚合机制
为了减少开销,802.11n提出帧汇聚技术。帧汇聚即把两个以上帧组合成一个帧进行传输。从而可以达到简化了帧结构，去除了以往协议帧之间的帧间间隔和竞争时间，从而提高了MAC层吞吐量。因为多个帧以单个进行传输，潜在冲突时间及回退时间大大减少。为了容纳这些大的汇聚帧，80211n最大帧长度也从4KB提高到64KB。
在传输双方在协商是否使用到帧聚合的格式，在管理帧中Beacon、Probe request、Probe response、Association request、Association response中HT信息里的A-MPDU参数字段，用于告知对方本端设备可以支持802.11n帧聚合传输。
![](\images\wifi\wifi_0002.png)
Maximum Rx A-MPDU Size指出站点可接收的最大长度的A-MPDU数据包，值可取0-3，可接收最大A-MPDU的长度为 2^(13 + Maximum A-MPDU Length Exponent)  - 1 octets.
Mimimum MPDU Start spacing指出站点可接收的在一个A-MPDU中相邻两个MPDU子帧发送的最小时间间隔，值可取0-7，含义如下：

0 for no restriction
1 for 1/4 us
2 for 1/2 us
3 for 1 us
4 for 2 us
5 for 4 us
6 for 8 us
7 for 16 us

MAC层的两种帧聚合机制,即MAC层业务数据单元聚合(MAC Service Data Unit Aggregation,A-MSDU)和MAC层协议数据单元聚合(MAC Protocol Data Unit Aggregation,A-MPDU)。分析结果表明,A-MSDU拥有较小的聚合开销但抗误码性能较差,对于不同的误码率,存在最优的聚合帧长使A-MSDU的吞吐量达到最大。A-MPDU虽然聚合开销较大,但却具有较好的抗误码性能。
![](\images\wifi\wifi_0008.png)
在进入MAC处理过程之前，所有的报文都以MSDU形式存在，经过MAC层处理之后，MSDU转换成MPDU。A-MSDU在进入MAC层之前完成聚合，而A-MPDU在MAC层之后完成聚合。
## 2.1 A-MSDU
80211协议栈收集一定数量的上层MSDU报文，先将其聚合，再转化为802.11MAC帧，即为MSDU聚合。
MSDU是指Ethernet报文，MSDU经过添加完整性校验MIC、分帧、省电模式下报文缓存、加密、序列号赋值、CRC校验、MAC头之后成为MPDU，MPDU就是指的经过802.11协议封装过的数据帧。

A-MSDU技术是指把多个MSDU通过一定的方式聚合成一个较大的载荷。通常，当AP或无线客户端从协议栈收到报文(MSDU)时，会打上Ethernet报文头，我们称之为A-MSDU Subframe；而在通过射频口发送出去前，需要一一将其转换成802.11报文格式。而A-MDSU技术旨在将若干个A-MSDU Subframe聚合到一起，并封装为一个802.11报文进行发送。从而减少了发送每一个802.11报文所需的PLCP Preamble，PLCP Header和802.11MAC头的开销，同时减少了应答帧的数量，提高了报文发送的效率。见下图：
![](\images\wifi\wifi_0005.png)
![](\images\wifi\wifi_0003.png)

A-MSDU报文是由若干个A-MSDU Subframe组成的，每个Subframe均是由Subframe header (Ethernet Header)、一个MSDU和0-3字节的填充组成。A-MSDU技术只适用于所有MSDU的目的端为同一个HT STA的情况。
一个A-MSDU由多个A-MSDU子帧所构成，每个A-MSDU子帧包括A-MSDU子帧头、可变长度的MSDU字段和0-3个字节的填充字段Padding。每个A-MSDU子帧（除了最后一个）都被填充为4个字节的整数倍。最后一个A-MSDU子帧没有填充字段。
A-MSDU子帧头仅仅包含了三个部分：DA、SA和长度。A-MSDU子帧头的目的站点地址和源站点地址部分字段的值分别对应的mac地址。长度字段包含了MSDU的长度（以字节为单位）。
由于聚合后的MSDU子帧没有独立的80211MAC帧头，这就要求所聚合的子帧的SA和DA所映射的TA和RA是相同的，及所聚合的帧由同一个无线端发送，由同一个无线端接收。聚合后，所有的子帧都会采用统一的加密方式，而且经聚合的帧将只能具备一种Qos属性，不允许将不同Qos属性的帧聚合。
A-MSDU的传送也存在许多限制。STA不能传送A-MSDU到不接收高速率传送的STA，即不接收包含HT Control字段的MAC帧的STA。如果一个STA的A-MSDU的长度超过了其对等实体所能接收的最大A-MSDU长度，则这个STA不能向此对等实体发送此A-MSDU聚合帧。
![](\images\wifi\wifi_0009.png)
Maximum A-MSDU length: 定义了最大可支持的A-MSDU的长度。当值为0，可支持最大长度为3839字节；当值为1，可支持长度为7935字节。

局限性：A-MSDU只适合用于较小的MSDU的聚合，由于当聚合帧较长，只有一个FCS，所以A-MSDU的传输可靠性较差。聚合A-MSDU的缺点在高误码率的信道中表现明显。所有的MSDU被聚合为一个MPDU，只被分配一个MAC序列号，若有一个MSDU子帧在传输过程中出现错误，整个A-MSDU将会被重传。
## 2.2 A-MPDU
与A-MSDU不同的是，A-MPDU聚合的是经过802.11报文封装后的MPDU，这里的MPDU是指经过802.11封装过的数据帧。通过一次性发送若干个MPDU，减少了发送每个802.11报文所需的PLCP Preamble，PLCP Header，从而提高系统吞吐量。见下图：
![](\images\wifi\wifi_0006.png)
![](\images\wifi\wifi_0004.png)

其中MPDU格式和802.11定义的相同，而MPDU Delimiter是为了使用A-MPDU而定义的新的格式。A-MPDU技术。同样只适用于所有MPDU的目的端为同一个HT STA的情况。
MPDU帧聚合是更低层次的聚合，80211协议栈首先收集一定数量的80211MAC帧，再将其进行聚合后打上PHY标头进行传输。MPDU聚合帧中的每个子帧均具有完整的80211MAC帧头。
![](\images\wifi\wifi_0010.png)
MPDU的每一个成分帧都有一个额外的80211帧头，在这一点上MPDU不如MSDU高效。这种对比在MPDU使用加密的时候更加明显，因为MPDU在每个成分帧上都加上了开销，而MSDU仅仅对一个汇聚帧加密，仅有单个开销。
一个A-MPDU有一个或多个A-MPDU子帧所构成。每个A-MPDU子帧有MPDU delimiter和MPDU实体组成。对于一个A-MPDU，除了最后一个A-MPDU子帧以外，其他A-MPDU子帧都会添加0-3个字节的填充字段，使子帧长度成为4字节的整数倍。对于一个A-MPDU中的所有MPDU，其MAC帧的Duration/ID字段都是相同的值。如果组成A-MPDU的MPDU子帧的最大长度为4095个字节，一个STA能传送长度为7935个字节的A-MSDU，那么此STA传送的组成一个A-MPDU的A-MSDU将被限制，从而是A-MSDU中的MPDU的长度小于4095个字节。
MPDU分界符的长度为4个字节。其格式与说明如下：
![](\images\wifi\wifi_0011.png)
MPDU分界符的目的是强制性的将聚合帧里的每个MPDU分开。强制的原因是因为当一个或多个MPDU分界符接收错误时，聚合帧的结构通常会被重新覆盖。每一个分界符与周围的MPDU具有相同的误码率，所以也会丢失。对于一个聚合帧里的所有MPDU，MAC帧头里的持续时间域取值相同。
![](\images\wifi\wifi_0012.png)
注意，一个MPDU长度为零，它的分界符依然有效。这样可以填充到MPDU之间以满足MPDU密度的限制要求或加在A-MPDU的最后。

# 3. 对聚合帧的块确认(BA)机制
为保证数据传输的可靠性，80211协议规定每收到一个单播数据帧，都必须立即回应以ACK帧。A-MPDU的接收端在收到A-MPDU后，需要对其中的每一个MPDU进行处理，因此同样针对每一个MPDU发送应答帧。Block Acknowledgement通过使用一个ACK帧来完成对多个MPDU的应答，以降低这种情况下的ACK帧的数量。
![](\images\wifi\wifi_0007.png)

**Block Ack机制分三个步骤来实现：**
(1)  通过ADDBA Request/Response报文协商建立Block ACK协定。
(2)  协商完成后，发送方可以发送有限多个QoS数据报文，接收方会保留这些数据报文的接收状态，待收到发送方的BlockAckReq报文后，接收方则回应以BlockAck报文来对之前接收到的多个数据报文做一次性回复。
(3)  通过DELBA Request报文来撤消一个已经建立的Block Ack协定。
![](\images\wifi\wifi_00013.png)
![](\images\wifi\wifi_00014.png)
对于BlockAck的确认分即时确认与延时确认两种。对于即时确认一般有硬件完成，对硬件收到的包映射到确认位图中，然后进行响应。而延时响应可以由协议栈完成，可以完成进一步的数据检查。对于流行的路由器一般只实现即时确认这种方式。通过Block Ack确认机制配合帧聚合的使用，大大提高在80211N的无线传输中的吞吐量。

上述ADDBA req和ADDBA resp两种帧里面，都有一个Buffer Size字段，即我们说的聚合包个数。标准对此的定义如下：
> The Buffer Size subfield indicates the number of buffers available for this particular TID. It is intended to provide guidance for the frame receiver to decide its reordering buffer size and is advisory only. When the AMSDU Supported field is equal to 0 as indicated by the STA transmitting the Block Ack Parameter Set field, each buffer is capable of holding a number of octets equal to the maximum size of an MSDU.

Bcm方案聚合包的个数可以通过wl ampdu_mpdu n加以设置，n即为发送时可以聚合的最大包个数，需要说明的是，这里设置的是最大聚合包个数，而不是限定必须的聚合包个数，实际发射的时候，聚合包个数可以等于n，也可以小于n，影响聚合包个数的因素有很大，比如上层业务流是否有稳定的业务需要发送、干扰因素、距离远近、每个PPDU最多可占用的信道时间NAV值，一般来说，设备的速率越高，（如带宽、天线数量、mcs等级等）允许聚合的包数也越大。


