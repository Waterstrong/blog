---
title: Jenkins Step by Step
date: 2016-07-06 21:30:44
category: Tools
tags: [DevOps, CI/CD, Jenkins]
description: Jenkins是一款开源的跨平台的可扩展的持续集成工具。作为目前使用最广泛，用户量最大的CI工具，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得较为出色，而且简单易学，入门上手快。
published: false
---

## Jenkins Introduction
Jenkins是一款开源的跨平台的可扩展的持续集成工具。作为目前使用最广泛，用户量最大的CI工具，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得较为出色，而且简单易学，入门上手快，当然Jenkins的优势还有很多，之前的项目上都一直在使用Jenkins，对于大多项目来说是完全满足条件的。
下图摘自官网，展示了持续交付场景下应用示例一般流程。
![](/assets/jenkins-step-by-step/pipeline_flow.png)

## Jenkins Installation
Jenkins由Java编写的开源产品，支持跨平台，安装也非常方便，可以到[Jenkins官网](https://jenkins.io/)下载需要的版本并安装。
![](/assets/jenkins-step-by-step/download.png)

以下大致列举出几种常用的安装和使用方式，在安装之前请确保已经安装了Java运行环境。

#### War包安装方式
可以直接下载自带Jetty的`*.war`包并运行，非常方便，比较推荐使用，Jenkins默认会运行在`8080`端口。
```
curl -O http://ftp.tsukuba.wide.ad.jp/software/jenkins/war-stable/2.7.1/jenkins.war
java -jar jenkins.war
```

也可以下载最新版本的[jenkins.war](http://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war)。

#### Docker镜像方式
可以直接下载Docker镜像来使用，无需进行安装，但需要有docker环境。
```
docker pull jenkinsci/jenkins
docker run -p 8080:8080 -p 50000:50000 -v /your/home:/var/jenkins_home jenkins
```

其中`/your/home`需要替换为你的主机路径，用于保存Jenkins的Workspace数据，更多细节可以参阅[Github jenkinsci](https://github.com/jenkinsci/docker)。

#### Ansible安装方式
另外，还可以通过Ansible自动化脚本安装，在另一篇博客[Ansible实践篇](/ansible-practice)中也有涉及。首先需要下载jenkins role到本地，假设下载到了`/usr/local/etc/ansible/roles/`目录下。
```
ansible-galaxy install geerlingguy.jenkins
```

然后编写`playbook`来实现**自动化批量安装**，假设创建一个playbook名为`setup_jenkins.yml`，其中`ci-server`是`inventory`文件中的Group或Host。
```
---
- hosts: ci-server
  become_method: sudo
  become: yes
  roles:
    - /usr/local/etc/ansible/roles/geerlingguy.jenkins
```

最后运行命令执行安装，稍等片刻后可访问主机的8080端口：
```
ansible-playbook -i inventory setup_jenkins.yml
```

如果需要设置Java版本可以在安装之前修改`geerlingguy.java`中默认的`java_packages`。更多说明可以参见[ansible-role-jenkins](https://github.com/geerlingguy/ansible-role-jenkins)和[ansible-role-java](https://github.com/geerlingguy/ansible-role-java)。

除了上述安装方式外，也可根据具体的操作系统进行安装，以下给出常用的操作系统下的安装方式。

#### 在RedHat/CentOS中安装
通常有两种安装方式：1. 添加Package Repository后使用yum进行安装和升级；2. 直接下载`*.rpm`进行安装。

**方式一：添加Package仓库安装**
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

**方式二：下载`*.rpm`进行安装**
目前最新版本是`2.7.1-1.1.noarch`，若需要指定安装版本，可到[http://pkg.jenkins-ci.org/redhat-stable/](http://pkg.jenkins-ci.org/redhat-stable/)查看并下载安装。
```
curl -O http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.7.1-1.1.noarch.rpm
sudo rpm -ivh jenkins-2.7.1-1.1.noarch.rpm
```

#### 在Ubuntu/Debian中安装
通常也有两种安装方式：1. 添加Debian Package Repository进行安装和升级；2. 通过直接下载`*.deb`进行安装。

**方式一：添加Package仓库安装**
```
wget -q -O - http://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
deb http://pkg.jenkins.io/debian-stable binary/
sudo apt-get update
sudo apt-get install jenkins
```

**方式二：下载`*.deb`进行安装**
目前最新版本是`2.7.1_all`，也可以到[http://pkg.jenkins-ci.org/debian-stable/](http://pkg.jenkins-ci.org/debian-stable/)查看并下载指定版本。
```
curl -O http://pkg.jenkins-ci.org/debian-stable/binary/jenkins_2.7.1_all.deb
sudo dpkg -i jenkins_2.7.1_all.deb
```

#### 在Mac或Windows下安装
对于Mac OS和Windows，直接下载对应安装包根据提示安装即可：[Mac OS安装包](https://jenkins.io/content/thank-you-downloading-os-x-installer/#stable)、[Windows安装包](https://jenkins.io/content/thank-you-downloading-windows-installer/#stable)。

## Jenkins Setup

#### System
#### Users
#### Email


## Jenkins Plugins


## Jenkins Jobs


## Jenkins Pipeline


## Jenkins Master and Slave


## Pipeline as Code

Jenkinsfile

**但Jenkins也有其缺点，比如：**
- 在配置Shell命令时，如果Pipeline规模扩大，构建和部署环境增多，那么就会复制粘贴很多这样的Shell命令，称为雪花式(Snowflakes)配置，增加维护成本；
- 另外，Jenkins定义Job的顺序是以Job为关注点，从全局出发，比如定义A Job的前置Job是B，后置Job是C，当Jobs顺序情况变得复杂就很难再梳理清楚了；
- Jenkins并未将Build Pipelines和Artifacts视作First-class Citizens，如果需要实现Continuous Delivery是需要借助插件完成，而Jenkins本身并不直接支持CD的；
- 此外，虽然Jenkins的插件生态系统管理得很好，一旦Workspace中有很多的插件，可能会有造成一些插件问题导致Build环境被污染的风险。
- 针对Jenkins 2.0的Jenkinsfile，可以将pipeline定义为代码形式，即Pipeline As Code，也方便了很多，算是优点，不过也是其缺点，最大的问题在于这可能造成在GUI上进行了修改而未修改Jenkinsfile的不一致性，而且无法追踪到这样的修改。



----
References

* [Jenkins Home](https://jenkins.io/)
* [Jenkins Tutorial](http://www.tutorialspoint.com/jenkins/index.htm)
* [Step by step guide to set up master and slave machines](https://wiki.jenkins-ci.org/display/JENKINS/Step+by+step+guide+to+set+up+master+and+slave+machines)
* [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)


