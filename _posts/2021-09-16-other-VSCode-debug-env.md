---
layout: article
title: "WSL+VSCode+python调试环境搭建"
data: 2021-09-11
tags:
  - python

---

## 摘要  

Windows Subsystem for linux VSCode的python编程调试环境的搭建

## VScode extension  

Remote-WSL  
Python
Remote-WSL安装完毕，再wsl系统中的相应代码目录中执行code .命令会trigger Windows系统中的vscode

## 调试的launch.json

如果要再vscode中能够进行调试工作，需要编写调试的launch.json如下，此处以调用pytest命令调试为例：  

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "PyTest",
            "type": "python",
            "request": "launch",
            "stopOnEntry": false,
            "pythonPath": "${config:python.pythonPath}",
            "module": "pytest",
            "args": [
                "-sv"
            ],
            "cwd": "${workspaceRoot}",
            "env": {},
            "envFile": "${workspaceRoot}/.env",
            "debugOptions": [
                "WaitOnAbnormalExit",
                "WaitOnNormalExit",
                "RedirectOutput"
            ]
        }
    ]
}
```
