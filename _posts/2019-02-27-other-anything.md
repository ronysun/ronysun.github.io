---
layout: article
title: "各种问题"
tags:
  - other
---
## git push 新的分支时提示错误

```bash
➜  libvirt git:(4.5.0) git push origin ctyun-4.5.0
error: src refspec ctyun-4.5.0 does not match any.
error: failed to push some refs to 'ssh://xx@172.18.211.200:29418/libvirt'
```

使用以下命令，成功提交：

```bash
 git push origin HEAD:ctyun-4.5.0
```

## iptables

添加多个端口，并添加注释

  ```bash
  iptables -I IN_public_allow -p tcp -m multiport --dport 2049,80 -j ACCEPT -m comment --comment "nginx&nfs"
  ```
