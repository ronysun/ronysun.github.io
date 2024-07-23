---
layout: article
title: "Switch Chip的评价体系"
data: 2020-11-20
tags:
  - CHIP

---

## 摘要

如何评价一个交换芯片的好与坏，应该从哪些维度去比较两个交换芯片的优劣？

## 性能指标

我们以Broadcom的Tomahawk 4的BCM56990和 Jericho2c+的BCM88850为例

- 交换容量：交换芯片能够进行多总输入流量的处理，56990的交换容量为25.6Tbps，即当有25.6Tbps流量同时进入交换芯片时，可以进行无阻塞的数据交换。
- 制造制程工艺水平：T2的制程为7nm，制程越小则单位面积内晶体管密度越小，会影响芯片的功耗和性能
- SerDes：目前高速网络接口均采用SerDes技术，56990支持512个SerDes
- 接口技术：接口技术决定交换芯片能够实现哪些接口，譬如50G PAM4接口能够实现10/25/50/100/200/400G接口。Tomahawk 4是512个50G PAM4 Serdes。

| 参数 | BCM56990 | BCM88850 | BCM56880 |
| --- | --- | --- | --- |
| 交换容量 | 25.6T | 7.2~9.6T | 12.8T |
| SerDes数量 | 512 | 144+192 | 256 |
| 制程工艺 | 7nm | 7nm | 7nm |
| 接口技术 | 50G PAM4 | 50G PAM4 | 50G PAM4 |
| 缓存 | 64MB | 64MB | 32MB |

## Broadcom公司的交换芯片

Broadcom公司的XGS家族包含Tomahawk和Trident两个系列，DNX家族包括Jericho系列。其中Tomahawk属于“傻快型”，带宽大，接口多，功能少，多用来做高端数据中心交换机；Trident属于“智商型”，功能特性多一些，多用来做企业级交换机；Jericho属于“城府型”，大缓存，可编程，多用来做城域汇聚路由器。  
![Braodcom交换芯片](https://5b0988e595225.cdn.sohucs.com/images/20191215/007f87b9118c4a11ac3420e54d154020.jpeg)

### Trident 4 BCM56880

Trident是L3 Programmable Ethernet Switch，支持NPL编程。256个50G PAM4 SerDes，交换容量为12.8Tb/s

### Tomahawk 2 BCM56970芯片

### Jericho2c+ BCM88850芯片

主要是面向路由器产品使用

- 内置MACSec和IPSec功能，实现对所有网络接口的进行线速加密
- 7.2Tbps的前面板I/O,9.6Tbps的fabric IO
- 前面板144*50Gbps SerDes，Cell-Based-Fabric interface 192*50Gbps SerDes
- The OCB is backed by two HBM2 stacks, which deliver an 8GB deep buffer operating at 2.4GT/s for 614GB/s
- 片上缓存为64MB，OCB后端有2个HBM2内存模块，提供8GB的深度缓存空间。在没有出现数据包拥塞的正常情况下，OCB模块起到数据包的缓存作用。一旦出现短期的数据拥塞，OCB模块无法缓存的数据包将会被转移到HBM2内存模块里。当OCB模块出现空余空间后，缓存在HBM2模块里的数据包又会被无缝地转移回来。这整个数据包转移的过程是自动进行的

-

## 参考

SerDes：在SerDes技术流行前，主要是并行总线接口。随着接口带宽的不断提高，由于信号质量等问题，并行总线接口提升的两种方式：“提升时钟频率”和“加大数据位宽”遇到了瓶颈。而SerDes采用的差分信号技术则完美避免了以上问题。Serializer把并行信号转化为串行信号。Deserializer把串行信号转化为并行信号。一般地，并行信号为8/10bit或者16/20bit宽度。编码则一般采用8/10B或。[SerDes详解](http://xilinx.eetrend.com/files-eetrend-xilinx/forum/201709/11981-32468-serdeszhi_shi_xiang_jie_.pdf)  
<http://www.shjinyuan.cn/engine/news/company//2020/1021/590.html>  
<https://www.sohu.com/a/360451846_258957>  
<https://www.nextplatform.com/2019/12/12/broadcom-launches-another-tomahawk-into-the-datacenter/>  
[深入了解思科交换机中的商用交换芯片](https://www.nextplatform.com/2018/06/20/a-deep-dive-into-ciscos-use-of-merchant-switch-chips/)  
