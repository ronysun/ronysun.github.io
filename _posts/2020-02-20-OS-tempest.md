---
layout: post
title: "Openstack的集成测试框架Tempst"
categories:
  - 测试

---
# 摘要

[官方文档](https://docs.openstack.org/tempest/latest/overview.html#)

## Quickstart

1. pip install tempest
2. tempest init cloud-01，此时会在当前目录下生成cloud-01目录，子目录中的etc包含tempest.conf和tempest.conf.sample。[tempest.conf的配置](https://docs.openstack.org/tempest/latest/configuration.html#tempest-configuration)

## tempest中的module

### abc

Python本身不提供抽象类和接口机制，要想实现抽象类，可以借助abc模块。ABC是Abstract Base Class的缩写。

### six

six是为了解决Python2 和 Python3 代码兼容性而产生的。six这个名字来源于 6 = 2 x 3，为什么不用‘Five’呢？5 = 2+3，一是因为乘法更有力量(more powerful)  

### stevedore

stevedore是用来实现动态加载代码的开源模块。它是在OpenStack中用来加载插件的公共模块。可以独立于OpenStack而安装使用。stevedore使用setuptools的entry points来定义并加载插件。entry point引用的是定义在模块中的对象，比如类、函数、实例等，只要在import模块时能够被创建的对象都可以。
