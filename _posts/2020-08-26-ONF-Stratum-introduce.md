---
layout: article
title: "Stratum introduce"
tags:
  - ONF

---
# Stratum introduce
## 摘要
Stratum是开源的芯片不相关（silicon-independent）的SDN交换机操作系统。其公开了一组SDN接口包括P4Runtime和OpenConfig/gNMI、gNOI。目前支持的设备包括Barefoot的Tofino、Broadcom的Tomahawk，以及bmv2软交换机。
[官方文档](https://www.opennetworking.org/stratum/)  
[代码库](https://github.com/stratum/stratum)

## Stratum Agent Architectural Components
[![StratumAgentArchitecturalComponents](https://github.com/stratum/stratum/raw/master/stratum/docs/images/stratum_architecture.png)]

## 接口介绍
1. P4 runtime：与支持P4编程的交换芯片通信，可配置可编程交换机上的数据面的转发通道
1. gNMI：gRPC Network Management Interface
1. gNOI：定义了一组基于gRPC的微服务，用于在网络设备上执行操作命令