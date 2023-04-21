---
layout: article
title: "各种问题"
tags:
  - other
---

## 一些git使用方法

```bash
git remote prune origin  #删除所有本地存在而远程库中已经删除掉的分支
git fetch --all --prune  #删除所有本地存在而远程库中已经删除掉的分支
git pull --rebase        #本地分支commit落后于远程分支，且本地有新提交，需要rebase远程分支
git push origin HEAD:ctyun-4.5.0  #向remote库推送一个新的分支
git push -u origin dev-2.5 --force #强制推送本地分支到远程分支，常用于本地执行了git commit --amend之后
```

## iptables

  ```bash
  iptables -I IN_public_allow -p tcp -m multiport --dport 2049,80 -j ACCEPT -m comment --comment "nginx&nfs"    #添加多个端口，并添加注释
  ```
