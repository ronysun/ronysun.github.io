---
layout: post
title: "EPC核心网介绍"
date: 2020-08-11
categories:
  - ONF
tags:
  - ONF

---
# 概述
EPC是Evolved Packet Core的缩写，是LTE网络核心网架构。
## 核心网架构图
![EPC](https://img-blog.csdn.net/20150909120950694)
### EPC中的网元
|网元|描述|
|---|---|
|UE|User Equipment|
|MME|Mobility Management Entity|
|HSS|Home Subscriber Server是一个database存储用户相关信息和订阅信息，提供用户移动性管理，用户验证，访问授权和会话管理|
|S-GW|Serving GW是radio side和ECP之间的连接点|
|P-GW|PDN(Packet Data Network) GW是ECP网络与外部IP网络之间的连接点|
### EPC中的接口
|接口|描述|
|----|----|
|S1-MME|eNodeB与MME之间的接口，运行SCTP和S1-AP协议|
|S1-U|eNodeB与S-GW之间的接口，运行GTP-U协议|
|S11|MME与S-GW之间的接口，运行GTP-C协议|
|S6a|MME与HSS之间的接口，运行SCTP协议及Diameter协议|
|S4|SGSN与S-GW之间的接口|
## LTE Protocol Stacks
### 用户面协议
![LTE用户面协议](https://img-blog.csdn.net/20150909135820709)
### 控制面协议
![LTE控制面协议](https://img-blog.csdn.net/20150909144404890)
## Traffic Flow on the LTE network
![Traffic Flow](https://github.com/ronysun/MarkdownImage/raw/master/LTE/TrafficFlowLTE.png)

## 参考
https://www.3gpp.org/technologies/keywords-acronyms/100-the-evolved-packet-core
https://www.cnblogs.com/soqu36/category/1749822.html

