---
layout: post  
title: Rally框架分析   
categories:  
  - openstack  
---
## rally/rally的目录结构
<pre>
.
├── __init__.py
├── aas
├── api.py
├── cli
├── common
├── consts.py
├── env
├── exceptions.py
├── plugins
├── task
├── ui
├── utils
└── verification
</pre>
### cli目录
<pre>
  .
├── __init__.py
├── cliutils.py
├── commands
│   ├── __init__.py
│   ├── db.py
│   ├── deployment.py
│   ├── env.py
│   ├── plugin.py
│   ├── task.py
│   └── verify.py
├── envutils.py
├── main.py
└── manage.py
</pre>

1. main.py提供了程序入口
   ```python
   categories = {
    "db": db.DBCommands,
    "env": env.EnvCommands,
    "deployment": deployment.DeploymentCommands,
    "plugin": plugin.PluginCommands,
    "task": task.TaskCommands,
    "verify": verify.VerifyCommands
    }
   ```
2. deployment负责认证信息和环境信息的管理：create,list,destroy,show,use...
3. task任务操作的方法包括：start，list，delete,report...
4. 以task.start的流程进行讲解：
   ```
   graph TD;
      A-->B;
      B-->C;
   ```

4. 各个方法会调用rally.api中对应的方法，如：task.start会调用_Task(APIGroup).start(),
5. 
### task目录
<pre>
  .
├── __init__.py
├── atomic.py
├── context.py
├── engine.py
├── exporter.py
├── functional.py
├── hook.py
├── processing
├── runner.py
├── scenario.py
├── service.py
├── sla.py
├── task_cfg.py
├── trigger.py
├── types.py
├── utils.py
└── validation.py
</pre>
1. 开发新的case的scenario类，需要继承scenario.py的Scenario基类
2. 开发新的context需要继承context.py的Context，而Context又继承BaseContext
3. 其他plugin基类还包括：sla，runner，hook
4. 以上基类都继承了rally.common.plugin中的plugin.Plugin
#### plugin基类
未完待续