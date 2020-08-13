---
layout: post
title: "各种问题"
categories:
  - 其他
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