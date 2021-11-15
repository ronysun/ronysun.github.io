---
layout: article
title: "SCTP introduce"
data: 2020-11-11
tags:
  - Network

---

## 摘要

SCTP是Stream Control Transmission Protocol的缩写。协议在RFC2960和3309中初始定义，现通过RFC4960更新定义。RFC4960被6096、6335、7053更新。本文档暂以4960为依据。

[RFC4960](https://tools.ietf.org/html/rfc4960)

## 缩略语

*chunk*:SCTP报文中的信息单元，由chunk头和chunk信息组成
*SCTP association*：SCTP端点之间的协议关系，由SCTP端点信息和协议状态信息组成，包括Verification Tag和当前active的一套Transmission Sequence Numbers（TSNs）组成

## SCTP会话

![SCTP会话](https://github.com/ronysun/MarkdownImage/raw/master/SCTP/SCTP-session.png)

## Verification Tag

verification tag是一个32bit的值，取值范围为1到2^32^。其作用主要是进行报文有效性验证，当SCTP的socket收到一个报文，其verification tag与预期值不一致时，则丢弃忽略该报文。一个SCTP的association包括进出两个方向会话方向的verification tag，其中入方向的verification tag是由INIT报文发送给对端，就是该报文中的initiate tag；出方向的verification tag则由INIT ACK报文中的initiate tag传递的回来。整个会话过程中，只有INIT报文的verification tag比较特殊为32bit的全0值。

## SCTP的多归属特性multi-homing

SCTP具备多归属特性，即SCTP会话的两个endpoint之间可以通过多个ip地址进行数据传输，当会话的primary path出现故障时，如一端接口down，则可通过其他path进行数据传输。当然当用户指定通过其他patch传输时，也可以手动切换到其他path。

## SCTP协议的测试

在ubuntu系统上安装lksctp-tools后，则有sctp_test工具
与iperf等测试工具类似，需要先开启一个服务端进程：sctp_test -H 127.0.0.1 -P 11011 -l
，再使用client端名称发送报文：sctp_test -H 127.0.0.1 -P 11012 -h 127.0.0.1 -p 11011 -s -x 5

## 参考

[SCTP报文](https://github.com/ronysun/MarkdownImage/raw/master/SCTP/sctp2.pcap)
