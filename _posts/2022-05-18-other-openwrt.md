---
layout: article
title: "OpenWRT开发"
data: 2022-05-18
tags:
  - develop

---

## 产品需求

1. wireguard
2. zerotier
3. tcpdump
4. 支持随身wifi模块上网
5. 端口映射
6. gost代理加密
7. BBR拥塞控制

可选项：

1. 远程桌面
2. 向日葵远程控制

## 信息  

硬件：  

| 项目 | 参数 |
| --- | --- |
| Model |FriendlyElec NanoPi R2S |
| Architecture | ARMv8 Processor rev 4 |

查看架构命令：opkg print-architecture

## 烧写官方固件

1. 下载官方固件21.02.3：<https://downloads.openwrt.org/releases/21.02.3/targets/rockchip/armv8/>
1. 使用balenaEtcher将OpenWRT烧写到TF卡中 <https://wiki.stationpc.cn/docs/stationpc/openwrt>
1. 更新opkg源<https://mirrors.ustc.edu.cn/help/openwrt.html>

   ```bash
   src/gz openwrt_core https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/targets/rockchip/armv8/packages
   src/gz openwrt_base https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/base
   src/gz openwrt_luci https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/luci
   src/gz openwrt_packages https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/packages
   src/gz openwrt_routing https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/routing
   src/gz openwrt_telephony https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/telephony
   ```

   或者

   ```bash
   sed -i 's_downloads.openwrt.org_mirrors.ustc.edu.cn/openwrt_' /etc/opkg/distfeeds.conf
   ```

## OpenWRT分区

### 调整overlay分区（Resizing partitions）

参考：<https://openwrt.org/docs/guide-user/installation/openwrt_x86#resizing_partitions>

1. 首先调整分区（Resizing partitions），通过cfdisk或fdisk将overlay分区resize，r2s中是将mmcblk0p2扩容
2. Resizing filesysstem，以resizing F2FS overlay举例

   ```bash
   opkg update
   opkg install losetup f2fs-tools
   LOOP="$(losetup -n -O NAME | sort | sed -n -e "1p")"
   ROOT="$(losetup -n -O BACK-FILE ${LOOP} | sed -e "s|^|/dev|")"
   OFFS="$(losetup -n -O OFFSET ${LOOP})"
   LOOP="$(losetup -f)"
   losetup -o ${OFFS} ${LOOP} ${ROOT}
   fsck.f2fs -f ${LOOP}
   mount ${LOOP} /mnt
   umount ${LOOP}
   resize.f2fs ${LOOP}
   reboot
   ```

### OpenWRT新增挂载分区

参考：<https://openwrt.org/docs/techref/block_mount>

1. 通过fdisk，cfdisk进行分区调整
2. 增加挂载点

   ```bash
   opkg update
   opkg install cfdisk block-mount
   ```

3. 安装block-mount后，在/etc/config/下出现fstab配置

   ```bash
   block info    #获取硬盘分区信息，uuid
   vim /etc/config/fstab   # 编辑分区挂载点配置,enable为1时，开机自动挂载
   ```

## OpenWRT中使用随身wifi

1. 需要添加一个端口eth2
2. 需要设置防火墙区域：将eth2接口发送到lan区域的“出站数据”设置为“接受”

## OpenWRT使用wireguard

OpenWRT使用wireguard有两种方式：

1. 添加wireguard接口
2.通过安装wireguard工具，编写wireguard配置文件后台操作 

```bash
mkdir /etc/wireguard
opkg update
opkg install kmod-udptunnel4 kmod-udptunnel6 kmod-wireguard wireguard-tools wireguard luci-proto-wireguard luci-app-wireguard unshare coreutils-stat
vim /etc/rc.local   #添加 wg-quick up wg0 
```

将[wg-quick](../assets/wg-quick)拷贝至/usr/sbin/

## zerotier配置

```bash
curl -s https://install.zerotier.com | sudo bash  #安装zerotier
service zerotier enable
```

修改/etc/config/zerotier文件中list join为自己的network ID

## OpenWRT中的gost

### 安装gost

官方网站：<https://gost.run/>

### 服务端配置

#### 配置

```json
{
    "Debug": false,
    "Retries": 0,
    "ServeNodes": [
        "relay+tls://0.0.0.0:10000"
    ]
}

```

#### 服务端服务文件

```bash
[Unit]
Description=GOST Service
After=network.target
Wants=network.target

[Service]
# This service runs as root. You may consider to run it as another user for security concerns.
# By uncommenting the following two lines, this service will run as user gost/gost.
# More discussion at https://github.com/gost/gost-core/issues/1011
# User=gost
# Group=gost
Type=simple
PIDFile=/run/gost.pid
ExecStart=/usr/bin/gost -C /etc/gost/config.json
Restart=on-failure
# Don't restart in the case of configuration error
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target

```

### 客户端

#### 客户端配置

路由端口转发功能，路由client端配置文件example，放置在OpenWRT上的/etc/myproxy目录下，命名为config.json:

```json
{
    "Debug": false,
    "Retries": 1,
    "ServeNodes": [
        "tcp://0.0.0.0:10001/DOMAIN:PORT",  //10001是本地服务端口，DOMAIN和PORT是目标地址和端口
        "tcp://0.0.0.0:10002/DOMAIN2:PORT2"
    ],
    "ChainNodes": [
        "relay+tls://SERVER-DOMAIN:PORT"  //服务端地址和端口
    ]
}

```

#### OpenWRT上gost作为client的服务脚本

放置在/etc/init.d/myproxy

```bash
#!/bin/sh /etc/rc.common
START=50

BIN=my-proxy
DAEMON=/usr/sbin/$BIN
DESC=$BIN
RUN_D=/var/run
CONFIG_FILE=/etc/myproxy/config.json
LOG_FILE=/var/log/myproxy.log

start() {
  echo -n "Starting $DESC "
  $DAEMON  -C $CONFIG_FILE > $LOG_FILE 2>&1 &
  echo "."
}

stop() {
  echo -n "Stopping $DESC "
  kill `pidof $BIN` > /dev/null 2>&1
  echo "."
}
```

## OpenWRT编译

系统：ubuntu 22.04
参考：<https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem>

```bash
sudo apt update
sudo apt install gcc make unzip bzip2 g++ lib32ncurses-dev  
git clone https://git.openwrt.org/openwrt/openwrt.git  # download source code
git checkout -b 21.02  # 切换到发布分支
./scripts/feeds update -a  #默认没有luci，需要执行更新操作
./scripts/feeds install -a
make menuconfig       # 编译配置
make                  # 开始编译
```

编译完成的结果可使用dd命令烧写到TF卡中

```bash
gunzip bin/targets/rockchip/armv8/$FILE #先解压结果
sudo dd if=bin/targets/rockchip/armv8/$FILE of=/dev/$device status=process #烧写至TF
```

### 修改默认密码

在package/base-files/files/etc/shadow中添加  
root:$1$NnC3ULCE$yzPuIIXiWbtQgg.ROhWpH1:16821:0:99999:7:::

### 增加文件

例如要在/usr/sbin下添加一个test.bin文件
则先创建对应目录：package/base-files/files/usr/sbin/
再将test.bin拷贝到该目录下，添加完成

### Reference

<https://www.moewah.com/archives/4003.html>

## 服务端搭建

Ubuntu 22.04 LTS

### 软件安装

 apt install rinetd ddclient

### rinetd配置

 ```bash
0.0.0.0 10000 www.baidu.com 5222

 ```

### ddclient配置

 ```bash
 
#use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com

use=web, web=https://api.ipify.org/
login=USERNAME
password=PASSWORD
DOMAIN
```
