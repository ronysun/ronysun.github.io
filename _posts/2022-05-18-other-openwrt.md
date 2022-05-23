---
layout: article
title: "OpenWRT开发"
data: 2022-05-18
tags:
  - other

---

## 信息  

硬件：
|项目|参数|
|---|---|
|Model|FriendlyElec NanoPi R2S|
|Architecture|ARMv8 Processor rev 4|

查看架构命令：opkg print-architecture

## 烧写官方固件

1. 下载官方固件21.02.3：<https://downloads.openwrt.org/releases/21.02.3/targets/rockchip/armv8/>
1. 使用balenaEtcher将openwrt烧写到TF卡中 <https://wiki.stationpc.cn/docs/stationpc/openwrt>
1. 更新opkg源<https://mirrors.ustc.edu.cn/help/openwrt.html>

   ```bash
   src/gz openwrt_core https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/targets/rockchip/armv8/packages
   src/gz openwrt_base https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/base
   src/gz openwrt_luci https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/luci
   src/gz openwrt_packages https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/packages
   src/gz openwrt_routing https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/routing
   src/gz openwrt_telephony https://mirrors.ustc.edu.cn/openwrt/releases/21.02.3/packages/aarch64_generic/telephony
   ```

## 调整overlay分区（Resizing partitions）

参考：<https://openwrt.org/docs/guide-user/installation/openwrt_x86#resizing_partitions>

1. 首先调整分区（Resizing partitions），通过fdisk将剩余分区新建
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

## openwrt新增挂载分区

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

## 安装软件

```bash
opkg update
opkg install nginx-all-module  #安装nginx
curl -s https://install.zerotier.com | sudo bash  #安装zerotier
```

### 安装gost

官方网站：<https://gost.run/>

## zerotier配置

1. service zerotier enable
1. 修改/etc/config/zerotier文件中list join为自己的network ID

## 配置gost

服务端配置：

```json
{
    "Debug": false,
    "Retries": 0,
    "ServeNodes": [
        "relay+tls://0.0.0.0:10000"
    ]
}

```

路由端口转发功能，路由client端配置文件example:

```json
{
    "Debug": false,
    "Retries": 1,
    "ServeNodes": [
        "tcp://0.0.0.0:10001/DOMAIN:PORT"  //14433是本地服务端口，DOMAIN和PORT是目标地址和端口
    ],
    "ChainNodes": [
        "relay+tls://SERVER-DOMAIN:PORT"  //服务端地址和端口
    ]
}

```

## nginx配置

1. 修改/etc/config/nginx文件关闭uci配置并删除其中的server配置
1. 新建/etc/nginx/nginx.conf：

   ```bash
   # This file is re-created when Nginx starts.
   # Consider using UCI or creating files in /etc/nginx/conf.d/ for configuration.
   # Parsing UCI configuration is skipped if uci set nginx.global.uci_enable=false
   # For details see: https://openwrt.org/docs/guide-user/services/webserver/nginx

   worker_processes auto;

   user root;

   events {}

   http {
           access_log off;
           log_format openwrt
                   '$request_method $scheme://$host$request_uri => $status'
                   ' (${body_bytes_sent}B in ${request_time}s) <- $http_referer';

           include mime.types;
           default_type application/octet-stream;
           sendfile on;

           client_max_body_size 128M;
           large_client_header_buffers 2 1k;

           gzip on;
           gzip_vary on;
           gzip_proxied any;

           root /www;

   }
   include conf.d/*.conf;

   ```

1. 新建/etc/nginx/conf.d/xxx.conf

   ```bash

   stream {

       upstream stream_backend {
            server example:6000;
       }

       server {
           listen                6000 ;
           proxy_ssl             on;
           proxy_pass            stream_backend;

           ssl_certificate       /root/cacert.pem;
           ssl_certificate_key   /root/privkey.pem;
           ssl_protocols         SSLv3 TLSv1 TLSv1.1 TLSv1.2;
           ssl_ciphers           HIGH:!aNULL:!MD5;
           ssl_session_timeout   4h;
           ssl_handshake_timeout 30s;
           #...
        }
   }
   ```

## 配置服务使能

service nginx enable  
service zerotier enable

## 服务端搭建

Ubuntu 22.04 LTS

### 软件安装

 apt install rinetd ddclient gost  
 rinetd配置  

 ```bash
0.0.0.0 10000 www.baidu.com 5222

 ```

 ddclient配置

 ```bash
 
#use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com

use=web, web=https://api.ipify.org/
login=USERNAME
password=PASSWORD
DOMAIN
```
