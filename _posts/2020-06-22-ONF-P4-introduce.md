---
layout: post
title: "P4 introduce"
categories:
  - OpenNetworking
tags:
  - OpenNetworking

---
# P4 introduce
## 摘要
开发环境：windows10 virtualBox vagrant  
[官方文档](https://www.opennetworking.org/p4/)  
[代码库](https://github.com/p4lang)

## P4中的一些概念
1. P4是协议不相关的，所谓协议相关就是只能根据现有的协议来进行编程，例如OpenFlow1.0时有12个字段（IP，MAC等），但这明显是远远不够的。那么P4就解决了这个问题
1. P4可以定义自己的匹配字段和动作（action），从而定义流表，进而形成流水线
1. PISA（Protocol Independent Switch Architecture），一种可编程网络芯片架构
1. P4 runtime的定位与OpenFlow一致，与P4配合使用
1. P4c是P4的编译器项目
1. behavioral-model（bmv2）是一个支持P4的软交换机version2
1. PI是P4 runtime的实现，用于控制面对数据面的控制
1. mininet可以构建虚拟网络，用于学习和测试

P4  target architecture
![P4 V1model](https://build-a-router-instructors.github.io/images/V1Model.png)
The V1Model consists of six P4 programmable components:
- Parser
- Checksum verification control block
- Ingress Match-Action processing control block
- Egress Match-Action processing control block
- Checksum update control block
- Deparser

## 开发环境部署
[Get Start](https://p4.org/p4/getting-started-with-p4.html)  
[官方提供的p4开发虚机镜像](https://drive.google.com/uc?id=1lYF4NgFkYoRqtskdGTMxy3sXUV0jkMxo&export=download)  
[国内某开发者的虚机镜像](https://share.weiyun.com/581m3WN)  
由于P4开发环境各个组件之间有依赖，且兼容性做的不好，所以不要单独升级某一组件到最新版，可能与其他组件不兼容
1. virtualbox创建Ubuntu16.04，desktop版，官方建议的版本，其他版本未经测试
1. 执行root-bootstrap.sh和user-bootstrap.sh，代码全部修改为gitee上的代码
1. 修改grpc中的submodule为gitee中的地址。修改.gitmodule，执行git submodule sync

## Firsh program  
参考 https://github.com/p4lang/tutorials/tree/master/exercises/basic
1. 在P4 tutorial的exercises/中有多个实例， 下是一个需要编程题（编写TODO部分），其子目录solution中的*.p4是添加了TODO部分的参考答案。
1. 在该basic目录下执行make run会执行p4c编译代码，并加载到ovs中，进入mininet进行测试。
1. 当basic.p4替换为solution中的文件时，各个host就可以ping通了。

## P4 language specification

### Overview
The core abstractions：

- Header type
- Parsers
- Tables
- Actions
- Match-action-unit
- Control flow
- Extern objects
- User-define metadata
- Intrinsic metadata

### Data plane interface

```
control MatchActionPipe<H>(in bit<4> inputPort,
                           inout H parsedHeaders,
                           out bit<4> outputPort);
```
此declaration描述了一个叫MatchActionPipe的block，该block通过对data-dependent sequence of match-action单元装置和其他必要的constructs进行programmed
- 第一个参数是一个命名为inputPort的4bit的值，其中的<span style="color:blue;">in<span>表示这是一个输入，不能修改
- 第二个参数是一个命名为parsedHeaders的H类型的object，<font color=Blue>inout</font>表明这个参数是both an input and an output
- 第三个参数是一个命名为outputPort的4bit的值，其中<font color=blue>out</font>表明这是一个初始值未定义的输出

### Extern objects and functions
P4 programs还能与体系结构（architecture）所提供的object and function交互（interact）。这些objects通过extern construct描述，such objects expose to the data-plane

一个extern object描述了一组由object实现的方法，但不是这些方法的实现（类似面向对象方法中的抽象类）下面这个结构体描述了the operations offered by an incremental checksum unit
```
extern Checksum16 {
    Checksum16();              // constructor
    void clear();              // prepare unit for computation
    void update<T>(in T data); // add data to checksum
    void remove<T>(in T data); // remove data from existing checksum
    bit<16> get(); // get the checksum for the data added since last clear
}
```
### Very Simple Switch Architecture（VSS）
1. Arbiter block
2. Parser runtime block
3. Demux block
4. Available exter blocks

### 变量类型
1. Boolean
2. integer
3. string

## 参考资料
1. https://p4.org/p4-spec/docs/P4-16-v1.2.0.html
1. https://www.sdnlab.com/22466.html
1. https://build-a-router-instructors.github.io/deliverables/p4-mininet/
1. https://www.eefocus.com/fpga/453687
