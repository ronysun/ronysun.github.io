---
layout: article
title: "K8S集群搭建"
data: 2021-05-05
tags:
  - CNCF
  - 云计算

---


## 摘要

基于CentOS7.9系统搭建

## 搭建K8S

### 安装kubeadm

使用国内aliyun源

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable kubelet
```

### 部署前配置

CentOS7需设置路由

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
swapoff -a  #kubernetes 1.8开始要求关闭系统的swap
```

```bash

wget -O /etc/yum.repos.d/docker.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y containerd.io  #安装containerd
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1 #设置转发
sysctl -w net.bridge.bridge-nf-call-iptables=1
```

### 部署K8S

```bash
kubeadm init --image-repository=registry.aliyuncs.com/google_containers  #使用aliyun源部署
kubeadm reset  #如中途部署失败，可通过reset重置
```

### 安装CNI插件

由于kubeadm不涉及CNI的安装，因此kubeadm部署完K8S集群后，该集群还不能真正可用，node状态处于NotReady，coreDNS处于pending
使用calico

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

### 添加节点

在其他节点执行[安装kubeadm](#安装kubeadm),[部署前配置](#部署前配置)，[安装CNI插件](#安装cni插件)，后执行：

```bash
kubeadm token create #先创建一个token
kubeadm join 172.25.0.6:6443 --token qkp5uf.6txn2b5si9rab0o9 --discovery-token-unsafe-skip-ca-verification #添加节点
kubectl label nodes k8s-node3 node-role.kubernetes.io/node3=
node/k8s-node3 labeled #添加角色标签
```


### 问题

kubeadm init前应保障kubelet服务正常运行，端口6443能够正常访问。kubelet未正常运行，有以下错误：

```bash
failed to get sandbox image \"registry.k8s.io/pause:3.6\": failed to pull image \"registry.k8s.io/pause:3.6\": failed to pull and unpack image \"registry.k8s.io/pause:3.6\": failed to resolve reference \"registry.k8s.io/pause:3.6\":
```

可通过以下两个方法解决

```bash
ctr -n k8s.io i pull k8s.gcr.io/pause:3.6  #如有代理可直接下载
ctr -n k8s.io image pull registry.aliyuncs.com/google_containers/pause:3.6 #先从国内源下载，后打tag 
ctr -n k8s.io image tag registry.aliyuncs.com/google_containers/pause:3.6 registry.k8s.io/pause:3.6
```

