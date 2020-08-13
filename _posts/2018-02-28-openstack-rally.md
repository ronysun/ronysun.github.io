---
layout: post
title: "rally"
categories:
  - openstack 
---
## rally命令
version,bash-completion,task,plugin,verify,deployment
### rally deployment
rally deployment常用命令：create, destroy, list, show, use
deployment 初始化环境常用的有两种方式：
1. source openrc, rally deployment create --fromenv --name=existing
2. rally deployment create --file=existing.json --name=existing   
[existing.json](https://github.com/openstack/rally/blob/master/samples/deployments/existing.json)

### rally task
rally task start samples/tasks/scenarios/nova/boot-and-delete.json
原生代码中已经存在很多测试场景的配置文件: [samples/tasks/scenarios](https://github.com/openstack/rally/tree/master/samples/tasks/scenarios)

### rally plugin
rally plugin list能够列出所支持的plugin。

### rally task templates
rally支持Jinja2的模板格式：
```yaml
---
  NovaServers.boot_and_delete_server:
    -
      args:
        flavor:
            name: "m1.tiny"
        image:
            name: "^cirros.*-disk$"
      runner:
        type: "constant"
        times: 2
        concurrency: 1
      context:
        users:
          tenants: 1
          users_per_tenant: 1

  NovaServers.resize_server:
    -
      args:
        flavor:
            name: "m1.tiny"
        image:
            name: "^cirros.*-disk$"
        to_flavor:
            name: "m1.small"
      runner:
        type: "constant"
        times: 3
        concurrency: 1
      context:
        users:
          tenants: 1
          users_per_tenant: 1
```
我们可以将上面的配置文件修改为：
```yaml
---
  NovaServers.boot_and_delete_server:
    -
      args:
        flavor:
            name: "m1.tiny"
        image:
            name: {{image_name}}
      runner:
        type: "constant"
        times: 2
        concurrency: 1
      context:
        users:
          tenants: 1
          users_per_tenant: 1

  NovaServers.resize_server:
    -
      args:
        flavor:
            name: "m1.tiny"
        image:
            name: {{image_name}}
        to_flavor:
            name: "m1.small"
      runner:
        type: "constant"
        times: 3
        concurrency: 1
      context:
        users:
          tenants: 1
          users_per_tenant: 1
```
修改配置文件后，可以通过两种方式传输image_name参数：
1. rally task start task.yaml --task-args 'image_name: "^cirros.*-disk$"'
2. rally task start task.yaml --task-args-file args.yaml ,yaml文件内容：
```yaml
---
  image_name: "^cirros.*-disk$"
```
