---
title: RabbitMQ入门指南
date: 2016-10-21 22:55:14
category: Frameworks
tags: [AMQP, RabbitMQ, Message, Topic, Exchange, Queue]
thumbnailImage: /assets/rabbitmq-guide/rabbitmq_logo.png
description: RabbitMQ是一款开源消息队列中间件，采用Erlang语言开发，RabbitMQ是AMQP的标准实现，在易用性、扩展性、高可用性等方面表现不错。
published: true
---

### RabbitMQ基本介绍
RabbitMQ(Rabbit Message Queue)，即消息队列系统，它是一款开源消息队列中间件，采用Erlang语言开发，RabbitMQ是[AMQP(Advanced Message Queueing Protocol)](https://spring.io/understanding/AMQP)的标准实现。

AMQP是一个公开发布的异步消息的规范，是提供统一消息服务的应用层标准**高级消息队列协议**，为面向消息的中间件设计，消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。相对于[JMS(Java Message Service)](https://en.wikipedia.org/wiki/Java_Message_Service)规范来说，JMS使用的是特定的APIs，而消息格式可自由定义，而AMQP对消息的格式和传输是有要求的，但实现不会受操作系统、开发语言以及平台等的限制。

JMS和AMQP还有一个较大的区别：JMS有队列(Queues)和主题(Topics)两种形式，发送到JMS队列的消息最多只能被一个Client消费，发送到JMS主题的消息可能会被多个Clients消费；AMQP只有队列(Queues)，队列的消息只能被单个接受者消费，发送者并不直接把消息发送到队列中，而是发送到Exchange中，该Exchage会与一个或多个队列绑定，能够实现与JMS队列和主题同样的功能。

RabbitMQ的主要宣传点在于其健壮性好、易于使用、高性能、高并发、集群易扩展以及强大的开源社区支持，鉴于些，在实际项目中使用还是比较可靠有价值的。接下来就介绍一下RabbitMQ从安装到简单示例的入门教程。

### RabbitMQ下载安装
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

### RabbitMQ入门使用

以Java为例集成RabbitMQ，针对不同应用场景使用其对应的功能，以下入门示例都出自[官网教程](http://www.rabbitmq.com/getstarted.html)。

##### 应用场景一：Simple Queue “Hello Word”
一个**P**roducer发送消息到Queue中，一个**C**onsumer从Queue读取消息并打印。
![](/assets/rabbitmq-guide/simple_queue.png)
有兴趣可参阅示例代码：[Hello World Demo](https://github.com/Waterstrong/spring-rabbitmq/tree/master/src/main/java/ws/demo/rabbitmq/message/helloWorld)。

##### 应用场景二：Work Queues
将消息分配给多个Consumer进行处理，可以避免在执行资源密集性任务时同步处理导致阻塞等待的问题，从而在一定程度上提升并行能力，通常称该类Consumer为Work，多个Work在后台接收分配到的任务并处理。
![](/assets/rabbitmq-guide/work_queues.png)

##### 应用场景三：Publish/Subscribe
前两个场景都是把一个消息传递给一个Consumer/Worker，而这里的Publish/Subscribe需要把消息传递给多个Consumer。这里Producer不会将消息直接发送到队列，事实上，Producer也并不知道消息会传递给任何的Queue，而是将消息发送到一个Exchange上，Exchange的作用在于收到Producer的消息并推送给绑定的Queue，这样Exchange就将消息传递给绑定的Queues及其以对应的Consumer了，这里使用的Exchange是`fanout`，其实就是广播功能。Exchange通常分为四种：
- fanout：该类型路由规则非常简单，会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，相当于广播功能
- direct：该类型路由规则会将消息路由到binding key与routing key完全匹配的Queue中，在场景四中会用到
- topic：与direct类型相似，只是规则没有那么严格，可以模糊匹配和多条件匹配，在场景五中会进一步解释
- headers：该类型不依赖于routing key与binding key的匹配规则来路由消息，而是根据发送的消息内容中的headers属性进行匹配

![](/assets/rabbitmq-guide/publish_subscribe.png)

比如一个打印Log的系统，当所有接收Log的Consumer接收到消息后都可以打印Log，有兴趣可参阅示例代码：[Publish/Subscribe Demo](https://github.com/Waterstrong/spring-rabbitmq/tree/master/src/main/java/ws/demo/rabbitmq/message/publishsubscribe)。

##### 应用场景四：Routing
在上一场景中，只是一个简单的Log系统，相当于广播功能，更进一步，可以针对不同的日志发送到不同的Consumer进行不同的处理，比如有的写文件，有的打印控制台等，那么就需要定义Routing路由了。为了使用Routing功能，可以将Exchange定义为`direct`类型，需要设置`routing_key`绑定到Queue上，然后就可以发送消息了。
![](/assets/rabbitmq-guide/routing.png)

##### 应用场景五：Topics
在以上场景中，使用`direct`类型的Exchange可以实现对不同消息的路由，但只是支持单一的条件，为了支持多种条件，比如不仅针对Log的级别，还要针对其来源进行分发消息，这时候可以使用Topic来实现了，其中的逻辑和`direct`的类似，只是`routing_key`支持多个用点`.`分隔的值用于匹配路由信息，其中路由Key可以使用`*`替代一个词，`#`匹配0个或多个词。
![](/assets/rabbitmq-guide/topics.png)

有兴趣可参阅示例代码：[Topics Demo](https://github.com/Waterstrong/spring-rabbitmq/tree/master/src/main/java/ws/demo/rabbitmq/message/topics)。

##### 应用场景六：RPC(Remote procedure call)
在前面的场景二中，能够实现使用*Work Queues*分发处理运算任务，但如果需要将任务发送到远程服务器上执行处理，然后等待返回运算结果呢？那就需要RPC远程过程回调了。这里描述的场景将和之前的完全不一样，需要构建一个Client和RPC Server，Client作为远程调用的发时携起者发送请求到`rpc_queue`上，RPC Server接收到请求后执行处理函数并将运算结果返回给`Callback Queue`，Client就可以接收到对应`correlationId`的返回结果了。
![](/assets/rabbitmq-guide/rpc.png)


### The End 结束语
本节只是针对RabbitMQ的主要功能及基本使用进行了介绍，如果对RabbitMQ的服务配置、客户端应用以及插件管理感兴趣，可以阅读下一篇博客[RabbitMQ进阶指南](/rabbitmq-prfessional)了解更多精彩内容。

----
References

* [RabbitMQ Home](http://www.rabbitmq.com/)
* [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)
* [Understanding AMQP](https://spring.io/understanding/AMQP)
* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)