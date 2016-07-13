---
title: Ansible基础篇
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

特别注意在使用命令行时会遇到当在`known_hosts`中新加入fingerprint时会弹出确认信息的问题，如果想要禁用确认，可以配置`/usr/local/etc/ansible/ansible.cfg`或`~/.ansible.cfg`并写入以下内容：
```
[defaults]
host_key_checking = False
```

或者直接在命令行中执行export命令：
```
export ANSIBLE_HOST_KEY_CHECKING=False
```
可以阅读更多关于[Host Key Checking](http://docs.ansible.com/ansible/intro_getting_started.html#host-key-checking)的介绍。接下来分别介绍一下CLI的常用指令：

#### $ ansible
ansible基本指令，用于ansible基本的操作，属于指令核心部分，其主要用于执行[Ad-Hoc](http://docs.ansible.com/ansible/intro_adhoc.html)命令，即单条命令。默认命令后需要跟主机和选项部分，默认不指定模块时，使用的是command模块。

以下为一些例子：
```
# ping all nodes
$ ansible all -m ping

# as bruce
$ ansible all -m ping -u bruce
# as bruce, sudoing to root
$ ansible all -m ping -u bruce --sudo
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce --sudo --sudo-user batman

# With latest version of ansible `sudo` is deprecated so use become
# as bruce, sudoing to root
$ ansible all -m ping -u bruce -b
# as bruce, sudoing to batman
$ ansible all -m ping -u bruce -b --become-user batman

# run a live command on all nodes
$ ansible all -a "/bin/echo hello"
```

#### $ ansible-playbook
ansible执行Playbook的命令，该指令使用最为频繁。
```
ansible-playbook -i inventory setup_server.yml
```

#### $ ansible-doc
该指令用于查看模块信息，常用参数有`-l`和`-s` ：
```
# 列出所有已安装的模块
$ ansible-doc -l

# 查看具体某模块，如command
$ ansible-doc -s command
```

#### $ ansible-galaxy
用于生成ansible最佳实践目录的命令，通常用于下载已经写好的roles，可以到[Ansible Galaxy](https://galaxy.ansible.com/)上搜索Roles，如`geerlingguy.jenkins`，然后安装。
```
ansible-galaxy install geerlingguy.jenkins
```

### Inventory
Inventory文件用来指定受控资源列表，也就是主机列表，可同时操作属于一个组的多台主机，组和主机之间的关系通过inventory文件配置。可以设置Hosts，指定Groups以及Groups中的Variables和Groups中的Groups。

#### Hosts and Groups
一个名为`hosts`的inventory文件例子，包括Hosts和Groups:
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

##### Host Variables
```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

##### Group Variables
```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

##### Groups of Groups, and Group Variables
```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

另外，可以查阅一些可用的Inventory参数[List of Behavioral Inventory Parameters](http://docs.ansible.com/ansible/intro_inventory.html#list-of-behavioral-inventory-parameters)。


### Modules
Module是Ansible中实际执行的命令，是具体任务的执行单元，可以理解为与系统中命令一一对应的模块. 如apt-get对应apt，wget对应get_url，分为Core Modules和Custom Modules。常用模块有：`ping`, `setup`, `file`, `command`, `shell`, `apt`以及`service`等。

#### ping
测试主机是否连通，Ping Module是Ansible的一个核心Modules之一，经常用来测试服务是否能连通，以及验证配置是否正确。
```
ansible -i hosts all -m ping
```

#### setup
Ansible有一些预定义的变量，定义了服务器的很多状态信息，可以使用setup动态获取，存储到本地的fact文件中供playbook调用。


#### file
主要用于远程主机上的文件操作的Module。


#### command
运行指定命令的Module。

#### shell
运行shell脚本，比如可直接在受控资源上执行的命令:
```
shell: ps -ef | grep jenkins
```

#### apt
用于安装软件的Module，即包管理器来管理软件包，对应于apt-get。

#### service
用于管理服务的Module


### Ansible与YAML
YAML是一种非常简单的数据描述语言，清晰易懂，利于阅读。YAML对缩进非常敏感，其中的基本数据类型有两种: Lists and Dictionaries。

Ansible Playbooks使用YAML语言，对使用者相对友好。Ansible中使用YAML需要注意的是：在Ansible中使用`""`来引用变量时，必须加引号，如：`with_items: "{{ packages }}"`。

若有兴趣可阅读更多关于[YAML Syntax](http://docs.ansible.com/ansible/YAMLSyntax.html)的介绍。

### Playbooks, Roles & Tasks
在Ansible中，最终被执行的自动化脚本叫做Playbooks，每个Playbook可能包含有多个Plays，每个Play可能包含有多个Tasks，每个Task是Ansible的最小执行单元，它都会利用相应的Module来执行对应的任务： `playbooks -> plays -> tasks -> modules -> variables`。

如果一个Playbook需要执行很多task，Playbook会变得非常庞大，而且其中的代码非常难以复用。通常Playbooks中会使用多个Roles来完成整个自动化任务，Role是Ansible代码复用的基本单位，它能够完整地实现一个独立任务。Role中会包含很多Tasks，每个Task是Ansible的最小执行单元。


## 如何使用Ansible?

可以移步笔者的Github[Ansible Workshop Step by Step](https://github.com/waterstrong/ansible-workshop)或另一篇博客[Ansible实践篇](/ansible-practice)来帮助一步步学习理解并使用Ansible，可以尝试一下，加深理解，主要包括以下几个步骤：

* [Step 1. Set up the environment](https://github.com/Waterstrong/ansible-workshop/blob/master/tutorials/STEP1.md)
* [Step 2. Inventory Practice](https://github.com/Waterstrong/ansible-workshop/blob/master/tutorials/STEP2.md)
* [Step 3. Playbooks, Roles and Tasks Practice](https://github.com/Waterstrong/ansible-workshop/blob/master/tutorials/STEP3.md)
* [Step 4. Install Apache2 Server Practice](https://github.com/Waterstrong/ansible-workshop/blob/master/tutorials/STEP4.md)
* [Step 5. Ansible Galaxy Practice](https://github.com/Waterstrong/ansible-workshop/blob/master/tutorials/STEP5.md)


以上内容就是对Ansible的基础介绍和入门部分，更多内容可参阅[官方网站](https://www.ansible.com/)。

----

### 参考资料

* [Ansible官方文档](http://docs.ansible.com/ansible/)
* [Ansible Workshop](https://www.gitbook.com/book/yaowenjie/ansible-workshop/details)
* [Ansible Training Workshop](https://github.com/richardzone/ansible-training-workshop)
