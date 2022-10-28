---
layout: article
title: "cobbler的使用"
data: 2022-08-09
tags:
  - other

---

## install

CentOS7.9系统

```bash

yum install epel-release
yum install cobbler cobbler-web pykickstart
setenforce 0   # 关闭selinux
cobbler check  # 初步检查
```

一些配置修改：

1. 修改server地址，提供cobbler服务的ip地址
2. 修改next_server地址，提供PXE服务的地址
3. 修改/etc/xinetd.d/tftp，diable改为"no"
4. 启动rsyncd服务，systemctl start rsyncd.service
5. 生成新的密码，openssl passwd -1 '123456'， 并修改/etc/cobbler/settings文件
6. 


## 参考

1. <https://cloud.tencent.com/developer/article/1180830>
1. cobbler web管理 <https://www.cnblogs.com/kittychentao2013/p/3311716.html>
