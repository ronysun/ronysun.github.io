---
layout: article
title: "SCTP introduce"
tags:
  - ONF

---
# SCTP introduce
## 摘要
SCTP是Stream Control Transmission Protocol的缩写。协议在RFC2960和3309中初始定义，现通过RFC4960更新定义。RFC4960被6096、6335、7053更新。本文档暂以4960为依据。

[RFC4960](https://tools.ietf.org/html/rfc4960)

## 缩略语
*chunk*:SCTP报文中的信息单元，由chunk头和chunk信息组成
*SCTP association*：SCTP端点之间的协议关系，由SCTP端点信息和协议状态信息组成，包括Verification Tag和当前active的一套Transmission Sequence Numbers（TSNs）组成
