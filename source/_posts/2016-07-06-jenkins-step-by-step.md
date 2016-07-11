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

![](jenkins-step-by-step/pipeline_flow.png)

## Jenkins Installation


### Jenkins Setup

### System
#### Users
#### Email


## Jenkins Plugins


## Jenkins Jobs


## Jenkins Pipeline


## Jenkins Master and Slave


## Pipeline as Code

Jenkinsfile

*但Jenkins也有其缺点，比如：*
- 在配置Shell命令时，如果Pipeline规模扩大，构建和部署环境增多，那么就会复制粘贴很多这样的Shell命令，称为雪花式(Snowflakes)配置，增加维护成本；
- 另外，Jenkins定义Job的顺序是以Job为关注点，从全局出发，比如定义A Job的前置Job是B，后置Job是C，当Jobs顺序情况变得复杂就很难再梳理清楚了；
- Jenkins并未将Build Pipelines和Artifacts视作First-class Citizens，如果需要实现Continuous Delivery是需要借助插件完成，而Jenkins本身并不直接支持CD的；
- 此外，虽然Jenkins的插件生态系统管理得很好，一旦Workspace中有很多的插件，可能会有造成一些插件问题导致Build环境被污染的风险。
- 针对Jenkins 2.0的Jenkinsfile方式可以将pipeline写成代码形式，但这可能造成在GUI上进行了修改而未修改Jenkinsfile的不一致性，而且无法追踪到这样的修改


----
References

* [Jenkins Home](https://jenkins.io/)
* [Jenkins Tutorial](http://www.tutorialspoint.com/jenkins/index.htm)
* [Step by step guide to set up master and slave machines](https://wiki.jenkins-ci.org/display/JENKINS/Step+by+step+guide+to+set+up+master+and+slave+machines)
* [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)


