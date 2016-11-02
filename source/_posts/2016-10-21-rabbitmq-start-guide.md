---
title: RabbitMQ入门指南
date: 2016-10-21 22:55:14
category: Frameworks
tags: [AMQP, RabbitMQ, Message, Topic, Queue]
thumbnailImage: /assets/rabbitmq-guide/rabbitmq_logo.png
description: RabbitMQ入门指南
published: true
---

## RabbitMQ基本介绍

优点缺点
应用场景
使用范围

## RabbitMQ下载安装
这里提供三种方式下载并安装RabbitMQ Server：
1. 需要在[RabbitMQ下载页](https://www.rabbitmq.com/download.html)下载对应操作系统的RabbitMQ Server安装包进行安装。

2. 通过命令行安装，比如如果在MAC系统下可以通过homebrew安装。
``` bash
$ brew install rabbitmq
$ rabbitmq-server       # 解压并以默认设置启动
```

3. 通过[Docker Compose](https://docs.docker.com/compose/)快速启动RabbitMQ Server，但需要保证已经安装并启动了Docker服务，然后创建`docker-compose.yml`的文件，并在同目录下执行命令`docker-compose up`即可启动RabbitMQ服务。推荐使用该方式，方便快捷，并在独立Container中运行。
``` yml docker-compose.yml
rabbitmq:
  image: rabbitmq:management
  ports:
    - "5672:5672"
    - "15672:15672"
```


## RabbitMQ入门使用

以Java为例集成RabbitMQ，并使用其各项功能，以下大部分入门示例都出自[官网教程](http://www.rabbitmq.com/getstarted.html)。

#### Simple Queue

#### Work Queues

#### Publish/Subscribe

#### Routing

#### Topics

#### RPC
RPC(Remote procedure call)


fanout, headers, topic, direct

Durability: Durable/Transient


## The End 结束语


----
References

* [RabbitMQ官网](http://www.rabbitmq.com/)
* [RabbitMQ教程](http://www.rabbitmq.com/getstarted.html)