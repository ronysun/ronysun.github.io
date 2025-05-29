---
layout: article
title: "DeepSeek介绍“
data: 2025-04-27
tags:
  - LLM
  - AI

---

## DeepSeek模型介绍

创新点：

- DeepSeek MoE
- MLA


## DeepSeek-V3 训练

### 训练方法创新

- experts负载均衡
- MTP(multi token predict)

### 并行方法

数据并行、专家并行、流水线并行、序列并行

### 训练成本

2k H800集群，训练58天。与Llama3.1-405B 16k H100集群，训练54天相比，训练成本仅为1/10。

训练成本下降的原因：

1. 混合精度
2. moe层计算与通信量下降（主要原因）

## DeepSeek-V3 推理

使用百卡集群进行推理，PD分离部署，Prefill 4机32卡，Decoding 40机320卡。根据SGlang团队的post，其采用PD分离架构，8卡H100服务器，3node作为Prefill节点，9node作为Decode节点，实现了2000输入输出序列时，input 52.3k tokens/s和22.3k output tokens/s的性能

## DeepSeek-V3 模型结构

| 配置项 | 参数  | 说明 |
| ---- | ---- | ---- |
| 总参数量 | 671B  | |
| 激活参数量 | 37B  | |


## References

<https://www.bilibili.com/video/BV18zcme1ELC>
<https://zhuanlan.zhihu.com/p/16323685381>
<https://lmsys.org/blog/2025-05-05-large-scale-ep/>