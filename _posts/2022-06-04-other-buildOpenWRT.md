---
layout: article
title: "OpenWRT编译开发"
data: 2022-06-07
tags:
  - other

---

## OpenWRT编译

系统：ubuntu 22.04
参考：<https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem>

```bash
sudo apt update
sudo apt install gcc make unzip bzip2 g++ lib32ncurses-dev  
git clone https://git.openwrt.org/openwrt/openwrt.git  # download source code
git checkout -b 21.02  # 切换到发布分支
make menuconfig       # 编译配置
make                  # 开始编译

```

### 添加文件

在

### 修改默认密码

在/etc/shadow中添加  
root:$1$NnC3ULCE$yzPuIIXiWbtQgg.ROhWpH1:16821:0:99999:7:::

### Reference

<https://www.moewah.com/archives/4003.html>