---
layout: article
title: "wireguard搭建和使用"
data: 2023-04-11
tags:
  - Network

---
## 服务端搭建

参考官网安装服务 <https://www.wireguard.com/install/>

## 生成配置

生成私钥，并根据私钥生成公钥
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey

在/etc/wireguard/目录下编写配置文件：

服务端配置

```bash
[Interface]
PrivateKey = <server-PrivateKey>
Address = 10.10.0.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE
ListenPort = 5000

[Peer]
PublicKey = <peer-client-PublicKey>
AllowedIPs = 10.10.0.2/32

[Peer]
PublicKey = <peer-client-PublicKey>
AllowedIPs = 10.10.0.3/32

```

客户端配置

```bash
[Interface]
Address = 10.10.0.3/32
PrivateKey = <>
DNS = 4.2.2.2

[Peer]
PublicKey = <对端publicKey>
AllowedIPs = 10.10.0.0/24, 10.8.8.0/24
Endpoint = server-ip:port
PersistentKeepalive = 16

```

## mesh组网

仅仅是client to client能够直连，非full mesh。

```bash
cat /proc/sys/net/ipv4/ip_forward  #检查服务器端是否开启了IP转发
sysctl -w net.ipv4.ip_forward=1    #开启IP转发
vim /etc/sysctl.conf    #将IP转发设置固化
```
