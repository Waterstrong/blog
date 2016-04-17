---
title: AWS EC2入门篇
date: 2016-04-11 12:00:08
category: DevOps
tags: [AWS, EC2, IaaS, 云服务, SSH]
description: Amazon EC2是一个IaaS云服务，主要提供弹性的计算资源，EC2是整个AWS最核心的组成部分，可以在很短的时间内创建、启动和运行不同的类型和大小的EC2实例。
---

### 基本介绍
#### 什么是Amazon EC2

> Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud.

Amazon EC2是一个IaaS云服务，主要提供弹性的计算资源，通俗地讲，就是提供多种类型的虚拟机。EC2也是整个AWS最核心的组成部分，AWS中有许多的服务需要依赖它。在EC2环境中，虚拟机被称为实例，实例的镜像被称为AMI(Amazon Machine Image)。使用AWS EC2有如下优势：

- 可避免前期的硬件投入，因此能够快速开发和部署应用程序
- 可根据自身需要快速启动任意数量的虚拟服务器、配置安全和网络以及管理存储
- 允许根据需要进行缩放以应对需求变化或流行高峰，降低流量预测需求
- 主要是根据类型和使用时间收费，即使用多少收多少的费用

#### 相关概念和名词解释

Instance: 实例，在EC2环境中，虚拟计算环境被称为实例。
AMI: Amazon Machine Image，亚马逊系统映像，即实例的预配置模板，其中包含服务器需要的程序包（包括操作系统和其他软件）。

IaaS: Infrastructure as a Service, 基础设施即服务。消费者通过Internet可以从完善的计算机基础设施获得服务，这类服务称为基础设施即服务(IaaS)，基于Internet的服务（如存储和数据库）是IaaS的一部分。

IaaS通常分为三种用法：公有云、私有云和混合云。Amazon EC2在基础设施云中使用公共服务器池(公有云)，更加私有化的服务会使用企业内部数据中心的一组公用或私有服务器池(私有云)，如果在企业数据中心环境中开发软件，那么这两种类型公有云、私有云都能使用(混合云)。

Internet上其他类型的服务包括平台即服务(Platform as a Service, PaaS)和软件即服务(Software as a Service, SaaS)。PaaS提供了用户可以访问的完整或部分的应用程序开发，SaaS则提供了完整的可直接使用的应用程序，比如通过 Internet管理企业资源。

###

1.EC2 concept & management


2.Instances management
3.Volumes 卷，硬盘， S3，EBS
df -h
df -T
sudo fdisk -l

sudo mkfs.ext4 /dev/xvdf

sudo mount /dev/xvdf /mnt/ebs

sudo umount /dev/xvdf


4.Snapshots 每个快照代表一个卷在一个特定时间点的状态。
5.Key Pairs 安全密钥对
6.Placement Groups
7.Elastic IPs 弹性IP，自己买的IP关联到Instance
8.Load Balancers

参考资料
[1] [Amazon EC2 User Guide for Linux Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
