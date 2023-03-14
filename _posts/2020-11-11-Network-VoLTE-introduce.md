---
layout: article
title: "VoLTE introduce"
data: 2020-11-11
tags:
  - Network

---

## 摘要

Voice over Long-Term Evolution或Voice over LTE，一般称高清通话，又译为长期演进语音承载。它基于IP多媒体子系统（IMS，IP Multimedia Subsystem）网络。


## 缩略语

*chunk*:SCTP报文中的信息单元，由chunk头和chunk信息组成
*SCTP association*：SCTP端点之间的协议关系，由SCTP端点信息和协议状态信息组成，包括Verification Tag和当前active的一套Transmission Sequence Numbers（TSNs）组成

## SIP协议

会话发起协议（Session Initiation Protocol）是一个由IETF MMUSIC工作组开发的协议，作为标准被提议用于创建，修改和终止包括视频，语音，即时通信，在线游戏和虚拟现实等多种多媒体元素在内的交互式用户会话。2000年11月，SIP被正式批准为3GPP信号协议之一，并成为IMS体系结构的一个永久单元。SIP与H.323一样，是用于VoIP最主要的信令协议之一。SIP报文内容发送会话描述协议（SDP），其描述了会话所使用流媒体细节，如：那个IP端口，采用哪种编解码器等等。RTP本身才是语音或视频的载体。

## 参考

<https://zh.wikipedia.org/wiki/>长期演进语音承载
