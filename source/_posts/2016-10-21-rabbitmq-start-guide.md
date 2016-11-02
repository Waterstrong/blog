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
RabbitMQ(Rabbit Message Queue)，即Rabbit消息队列，它是一款开源消息队列系统，采用Erlang语言开发，RabbitMQ是[AMQP(Advanced Message Queueing Protocol)](https://spring.io/understanding/AMQP)的标准实现。

AMQP是一个公开发布的异步消息的规范，是提供统一消息服务的应用层标准**高级消息队列协议**，为面向消息的中间件设计，相对于[JMS(Java Message Service)](https://en.wikipedia.org/wiki/Java_Message_Service)规范来说，JMS使用的是特定的APIs，而消息格式可自由定义，而AMQP对消息的格式和传输是有要求的，但实现不会受操作系统、开发语言以及平台等的限制。

JMS和AMQP还有一个较大的区别：JMS有队列(Queues)和主题(Topics)两种形式，发送到JMS队列的消息最多只能被一个Client消费，发送到JMS主题的消息可能会被多个Clients消费；AMQP只有队列(Queues)，队列的消息只能被单个接受者消费，发送者并不直接把消息发送到队列中，而是发送到Exchange中，该Exchage会与一个或多个队列绑定，能够实现与JMS队列和主题同样的功能。

RabbitMQ的主要宣传点在于其健壮性好、易于使用、高性能、高并发、集群易扩展以及强大的开源社区支持，鉴于些，在实际项目中使用还是比较可靠有价值的。接下来就介绍一下RabbitMQ从安装到简单示例的入门教程。

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

在安装完成并启动服务后，可通过访问[http://localhost:15672](http://localhost:15672)测试，默认用户名和密码是`guest`。

## RabbitMQ入门使用

以Java为例集成RabbitMQ，针对不同的应用场景使用其对应的功能，以下大部分入门示例都出自[官网教程](http://www.rabbitmq.com/getstarted.html)。

#### 应用场景一：Simple Queue “Hello Word”

#### 应用场景二：Work Queues

#### 应用场景三：Publish/Subscribe

#### 应用场景四：Routing

#### 应用场景五：Topics

#### 应用场景六：RPC(Remote procedure call)


fanout, headers, topic, direct

Durability: Durable/Transient


## The End 结束语
本节只是针对RabbitMQ的主要功能及基本使用进行了介绍，如果对RabbitMQ的服务配置、客户端应用以及插件管理感兴趣，可以阅读下一篇博客[RabbitMQ进阶指南](/rabbitmq-prfessional)了解更多精彩内容。

----
References

* [RabbitMQ Home](http://www.rabbitmq.com/)
* [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)
* [Understanding AMQP](https://spring.io/understanding/AMQP)
* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)