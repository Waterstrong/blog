---
title: Ansible入门与实践
date: 2016-06-26 19:56:40
category: Tools
tags: [DevOps, Ansible, 脚本, 自动化]
description: Ansible是一个IT自动化工具（IT Automation Tool）。它能够很容易地实现管理配置、软件部署、编排任务(如持续部署)等自动化运维工作。
---

## 什么是Ansible?
Ansible是一个IT自动化工具（IT Automation Tool）。它能够很容易地实现管理配置、软件部署、编排任务(如持续部署)等自动化运维工作。其主要目标就是变得更加简单易使用，同时也考虑到安全和可靠性等。

## 为什么要使用Ansible?
### 存在的问题及解决方案
当需要对系统配置进行管理、维护以及部署时通常采用文档方式或Shell脚本。对于上百上千台机器集群进行管理时文档已经不适用，而Shell方式对维护人员要求较高，而且容易出错。总得来说传统方式会存在以下几个方面的问题：
* 机器数量庞大，难于手动管理
* 大量的重复运维工作，浪费人力
* 复杂的系统，难于避免出错
* 不可跨平台，难以复用和维护

因此需要一个统一的管理工具，而且相对来说简单易用，而且能够支持跨平台，高可读性易于维护，高重用性提升效率，总之能够快速有效地完成自动化运维工作。而Ansible是众多自动化工具中较为出色的一款。

### 自动化配置管理工具对比
|Tools|DSL|Description|
|-----|:-:|:----------|
|ANSIBLE|YAML|Python，可维护性较高，架构简单，Windows平台支持有限|
|SALTSTACK|SLS，支持YAML|Python，简单快速灵活，Windows平台支持有限，大规模多功能支持欠缺|
|PUPPET|Ruby|Ruby，成熟度高，学习曲线较陡|
|CHEF|Ruby|Ruby，成熟度高，学习曲线较陡|

## Ansible基本概念
### Ansible CLI
Ansible CLI 包含以下几个常用的指令：`ansible`, `ansible-playbook`, `ansible-doc`以及`ansible-galaxy`。
