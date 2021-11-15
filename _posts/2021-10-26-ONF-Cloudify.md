---
layout: article
title: "Cloudify"
data: 2021-10-26
tags:
  - 云计算

---
# Cloudify

## 摘要  

[Cloudify](https://cloudify.co/) is an open source, multi-cloud orchestration platform，同时也是开发该系统的公司（该公司位于以色列的特拉维夫）的名称。通过集成AWS，Azure，OpenStack，Kubernetes，从而实现Speed Up Deployments, Optimize Costs, Ensure Compliance的愿景。其提供Blueprint可实现application在多云上的可靠部署。  
[TOSCA](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=tosca)（Topology and Orchestration Specification for Cloud Applications）是由OASIS组织制定的云应用拓扑编排规范。  
Cloudify的blueprint是基于TOSCA规范的

## 典型应用场景

基础架构的创建、更新
交付
IT自动化运维工作编排
其他场景

## [GetStart](https://docs.cloudify.co/latest/trial_getting_started/set_trial_manager/download_community/?submissionGuid=bcdff218-da28-447e-87ec-b313873375fc)

1. 通过docker启动一个all in one的测试环境，用户名密码为：admin：  

   ```bash
   sudo docker run --name cfy_manager_local -d --restart unless-stopped -v /sys/fs/cgroup:/sys/fs/cgroup:ro --tmpfs /run --tmpfs /run/lock --security-opt seccomp:unconfined --cap-add SYS_ADMIN -p 80:80 -p 8000:8000 cloudifyplatform/community-cloudify-manager-aio:latest
   ```

1. 上传Blueprint package： [link](https://github.com/cloudify-community/blueprint-examples/releases/download/latest/simple-hello-world-example.zip)
1. Deploy & install “hello world” application
1. 访问127.0.0.1:8000 可看到hello world页面（8000端口是第一步启动docker时做了映射的接口，且Blueprint中有此定义）

## Cloudify与Kubernetes

1. Cloudify可以为K8S提供IaaS层面的支持，allocating compute,network,storage resources.（AWS，Azure，OpenStack等）
1. Cloudify Defined by TOSCA或K8S原生的YAML文件方式， As a Workload Provisioner of K8S
1. Cloudify As a K8S Provisioner, 可以在多云上管理多个K8S集群
1. Cloudify可作为服务代理，从K8S集群内部访问外部资源  
   ![kubernetes-cloudify-integration](https://gitee.com/ronysun/images/raw/master/kubernetes-cloudify-integration.png)

## How Cloudify Enhances Helm’s Capabilities

   ![onap-helm-cloudify-integration](https://gitee.com/ronysun/images/raw/master/onap-helm-cloudify-integration.png)

## Bibliography

   [Cloudify Support For Helm](https://cloudify.co/blog/cloudify-support-for-helm-the-kubernetes-package-manager/)
