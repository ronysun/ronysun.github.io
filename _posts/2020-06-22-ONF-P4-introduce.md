---
layout: article
title: "P4 introduce"
tags:
  - ONF

---
# P4 introduce
## 摘要
P4是Programming Protocol-independent Packet Processors的缩写，
开发环境：windows10 virtualBox vagrant  
[官方文档](https://www.opennetworking.org/p4/)  
[代码库](https://github.com/p4lang)

## P4中的一些概念
1. P4是协议不相关的，所谓协议相关就是只能根据现有的协议来进行编程，例如OpenFlow1.0时有12个字段（IP，MAC等），但这明显是远远不够的。那么P4就解决了这个问题
1. P4可以定义自己的匹配字段和动作（action），从而定义流表，进而形成流水线
1. PISA（Protocol Independent Switch Architecture），一种通用的网络设备架构
1. P4 runtime的定位与OpenFlow一致，与P4配合使用
1. P4c是P4的reference编译器项目
1. behavioral-model（bmv2）是一个支持P4的软交换机version2
1. PI是P4 runtime的实现，用于控制面对数据面的控制
1. mininet是一个轻量的网络仿真环境，可以构建虚拟网络，用于学习和测试
1. P4 tagets指包括可编程ASIC，ovs，Bmv2等可加载P4程序的网络器件

一个简单的P4 Bmv2的simple switch target architecture
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

## First program  
参考 https://github.com/p4lang/tutorials/tree/master/exercises/basic
1. 在P4 tutorial的exercises/中有多个实例， 下是一个需要编程题（编写TODO部分），其子目录solution中的*.p4是添加了TODO部分的参考答案。
2. 在该basic目录下执行make run会执行p4c编译代码，并在mininet中执行Bmv2。该目录下的Makefile将pod-topo目录下的topology.json作为拓扑信息参数，传递给../../utils目录下的Makefile，并执行该目录下的run_exercise.py脚本。
3. 把basic.p4替换为solution中的文件时，各个host就可以ping通了。  
以basic.p4进行说明：  

```c
/*************************************************************************
*********************** H E A D E R S  ***********************************
*************************************************************************/
typedef bit<9>  egressSpec_t;
typedef bit<48> macAddr_t;
typedef bit<32> ip4Addr_t;

header ethernet_t {         //header数据结构对应packet中的header
    macAddr_t dstAddr;
    macAddr_t srcAddr;
    bit<16>   etherType;
}

struct headers {              //struct类似于python中的dictionary
    ethernet_t   ethernet;
    ipv4_t       ipv4;
}
/*************************************************************************
*********************** P A R S E R  ***********************************
*************************************************************************/
parser MyParser(packet_in packet,
                out headers hdr,
                inout metadata meta,
                inout standard_metadata_t standard_metadata) {

    state start {                   //进入parse过程，进入parse_ethernet
        transition parse_ethernet;
    }

    state parse_ethernet {
        packet.extract(hdr.ethernet);       //参数hdr的ethernet头，解析成功，header的validity会自动设置为true，即自动帮我们确定header的格式是否正确
        transition select(hdr.ethernet.etherType) {     //根据hdr.ethernet.etherType的值选择处理流程
            TYPE_IPV4: parse_ipv4;
            default: accept;
        }
    }

    state parse_ipv4 {
        packet.extract(hdr.ipv4);
        transition accept;
    }

}

/*************************************************************************
**************  I N G R E S S   P R O C E S S I N G   *******************
*************************************************************************/

control MyIngress(inout headers hdr,
                  inout metadata meta,
                  inout standard_metadata_t standard_metadata) {
    action drop() {
        mark_to_drop(standard_metadata);
    }
    
    action ipv4_forward(macAddr_t dstAddr, egressSpec_t port) {         //P4中的action就是C语言中的函数，此action需传入两个参数，此处可结合s1中ipv4_lpm这个table去理解
        standard_metadata.egress_spec = port;
        hdr.ethernet.srcAddr = hdr.ethernet.dstAddr;
        hdr.ethernet.dstAddr = dstAddr;
        hdr.ipv4.ttl = hdr.ipv4.ttl - 1;
    }
    
    table ipv4_lpm {            //建立一个名称为ipv4_lpm（longest prefix match）的table
        key = {
            hdr.ipv4.dstAddr: lpm;          //通过ipv4的dstAddr进行lpm
        }
        actions = {             //包含table中所有可能的动作
            ipv4_forward;
            drop;
            NoAction;
        }
        size = 1024;            //table的entry数量
        default_action = drop();
    }
    
    apply {
        if (hdr.ipv4.isValid()) {
            ipv4_lpm.apply();
        }
    }
}

/*************************************************************************
***********************  S W I T C H  *******************************
*************************************************************************/

V1Switch(
MyParser(),
MyVerifyChecksum(),
MyIngress(),
MyEgress(),
MyComputeChecksum(),
MyDeparser()
) main;
```
### S1交换机
在该实验中，通过静态建表的方式，在switch中的有MyIngress.ipv4_lpm表内容如下：
![ipv4_lpm_table](https://gitee.com/ronysun/ronysun/raw/master/image/P4introduce-ipv4-lpm-table.png)
则当packet的ipv4.dstAddr匹配table中match的key，根据命中的action（ipv4_forward），传入参数dstAddr和port,并进行处理
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

```c
control MatchActionPipe<H>(in bit<4> inputPort,
                           inout H parsedHeaders,
                           out bit<4> outputPort);
```
此declaration描述了一个叫MatchActionPipe的block，该block通过对data-dependent sequence of match-action单元装置和其他必要的constructs进行programmed
- 第一个参数是一个命名为inputPort的4bit的值，其中的<span style="color:blue;">in</span>表示这是一个输入，不能修改
- 第二个参数是一个命名为parsedHeaders的H类型的object，<span style="color:blue;">inout</span>表明这个参数是both an input and an output
- 第三个参数是一个命名为outputPort的4bit的值，其中<span style="color:blue;">out</span>表明这是一个初始值未定义的输出

### Extern objects and functions
P4 programs还能与体系结构（architecture）所提供的object and function交互（interact）。这些objects通过extern construct描述，such objects expose to the data-plane

一个extern object描述了一组由object实现的方法，但不是这些方法的实现（类似面向对象方法中的抽象类）下面这个结构体描述了the operations offered by an incremental checksum unit

```c
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
