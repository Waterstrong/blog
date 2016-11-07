---
title: RabbitMQ项目实践应用 - 发布订阅模式
date: 2016-11-01 23:26:31
category: Frameworks
tags: [AMQP, RabbitMQ, Message, Topic, Exchange, Queue]
thumbnailImage: /assets/rabbitmq-guide/publish_thumbnail.png
description: 当前有一个源系统G，主要存储大量数据，每条数据以唯一的ID标识，该系统每天会不定时处理一些合并数据的操作，出于某些需求原因，同时还有若干下游子系统A。
published: true
---

### 一、需求描述
**背景**
*当前有一个源系统G，主要存储大量数据，每条数据以唯一的ID标识，该系统每天会不定时处理一些合并数据的操作，出于某些需求原因，同时还有若干下游子系统A，其中保存了源系统数据的主键ID。*

**需求**
- 当源系统G发生了合并操作时需要通知所有下游子系统对相同ID的数据进行相应的更改
- 下游子系统数量不定，且可能在以后会逐渐增加，应尽可能降低源系统和子系统间的耦合
- 特别是在增加下游子系统时不需要修改源系统的代码就可完成通知的功能

### 二、解决方案
考虑到解耦，比较理想的方案是采用消息机制来实现，可以通过将源系统G的合并操作转化为消息发送给一个Exchange，然后由Exchange分发给每个下游子系统对应的Queue，由子系统各自独立处理并维护队列消息，这样，当增加一个子系统时，只需要添加一个Queue并绑定到Exchange上即可，很符合发布订阅模式，可以参阅之前博客 [RabbitMQ入门指南](/rabbitmq-start-guide) 中的场景三。
![](/assets/rabbitmq-guide/publish_subscribe.png)

### 三、方案实施
##### 管理RabbitMQ服务
在完成安装RabbitMQ后即可开始使用，这里采用虚拟机中Docker启动服务的方式，当前的虚拟机IP地址为`192.168.56.105`，在后续的配置和访问中都会用到。通常在程序中用代码实现创建并绑定Exchange和Queue，但为了将Queue的管理和访问权限分离开来，这里采用事先创建Exchange和Queue，然后程序只负责配置连接访问对应的Exchange和Queue的方式。如果对RabbitMQ的相关配置和应用还不太了解，可以参阅另一篇博客 [RabbitMQ进阶指南](/rabbitmq-professional)。
1. 首先登录到[http://192.168.56.105:15672](http://192.168.56.105:15672)，默认用户名和密码为`guest`。
![](/assets/rabbitmq-guide/overview_totals2.png)

2. 选择【channel】页，创建多个Queues，分别命名为`DEV.MESSAGE.QUEUE.APP1`, `...APP2`和`...APP3`。
![](/assets/rabbitmq-guide/queues.png)

3. 选择【exchanges】页，创建`topic`类型的Exchange，命名为`DEV.MESSAGE.TOPIC.MAIN`。
![](/assets/rabbitmq-guide/exchanges.png)

4. 选择并点击【DEV.MESSAGE.TOPIC.MAIN】条目，进入详细页面，选择【Bindings】，将步骤2中的Queues绑定到该Exchange上。
![](/assets/rabbitmq-guide/exchanges_topic.png)

这样，Exchange和Queue的创建及绑定工作就完成了，接下来需要分别完成发送者和接收者的代码。

##### 源系统发送消息
发送消息需要配置`ConnectionFactory`关联到RabbitMQ服务，然后通过`RabbitTemplate`发送消息到连接的Exchange中，这里选取`topic`类型作为示例讲解，如果只是单纯的广播，采用最基本的`fanout`类型也可以满足当前需求的，请根据项目的实际情况选用Exchange类型。
``` java RabbitMqConfiguration.java
package ws.message.configuration;
import org.springframework.amqp.rabbit.connection.AbstractConnectionFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.rabbitmq.client.ConnectionFactory;

@Configuration
public class RabbitMqConfiguration {

    @Value("${rabbitmq.host}")
    private String topicHost;

    @Value("${rabbitmq.username}")
    private String username;

    @Value("${rabbitmq.password}")
    private String password;

    @Value("${rabbitmq.port:5672}")
    private int port;

    @Value("${rabbitmq.virtualHost:/}")
    private String virtualHost;

    @Value("${rabbitmq.connectionTimeout}")
    private int connectionTimeout;

    @Value("${rabbitmq.recoveryInterval}")
    private long recoveryInterval;

    @Bean
    public AbstractConnectionFactory connectionFactory() {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(topicHost);
        factory.setUsername(username);
        factory.setPassword(password);
        factory.setPort(port);
        factory.setVirtualHost(virtualHost);
        factory.setConnectionTimeout(connectionTimeout);
        factory.setNetworkRecoveryInterval(recoveryInterval);
        return new CachingConnectionFactory(factory);
    }
}
```

在默认的`resources`目录下创建`application.yml`并添加需要的Properties，需要指定`host`、`username`、`password`以及`topicName`等配置参数，另外，`recoveryInterval`表示网络恢复重试的时间间隔，`connectionTimeout`表示连接超时，都以毫秒为单位。
``` yml application.yml
server:
  port: 8081
  context-path: /producer

rabbitmq:
  host: 192.168.56.105
  username: guest
  password: guest
  topicName: DEV.MESSAGE.TOPIC.MAIN
  recoveryInterval: 5000
  connectionTimeout: 10000
```

最后创建`MessageController`来模拟源系统的发送消息行为，采用`RabbitTemplate`将消息发送到指定的Topic上。
``` java MessageController.java
package ws.message.controller;
import static java.lang.String.format;
import static org.apache.log4j.Logger.getLogger;
import static org.joda.time.DateTime.now;
import static org.springframework.http.HttpStatus.CREATED;
import static org.springframework.web.bind.annotation.RequestMethod.POST;
import java.io.IOException;
import org.apache.log4j.Logger;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MessageController {
    private final static Logger LOGGER = getLogger(MessageController.class);

    @Value("${rabbitmq.topicName}")
    private String topicName;

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @RequestMapping(method = POST, value = "/messages")
    public ResponseEntity<?> sendMessage(@RequestBody String message)
            throws IOException {
        LOGGER.info(format("@%s Send message: %s", now(), message));
        rabbitTemplate.convertAndSend(topicName, "", message);
        return new ResponseEntity<>(message, CREATED);
    }
}
```

完整代码可以参考GitHub Demo: [message-producer](https://github.com/Waterstrong/message-producer)。

##### 子系统接收消息
需要配置连接的RabbitMQ服务和特定的Queue，该Queue是绑定在源系统的Exchange上的，同时需要设置消息监听者`RabbitMqConsumer`。
``` java RabbitMqConfiguration.java
package ws.message.configuration;
import java.net.URI;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import ws.message.consumer.RabbitMqConsumer;

@Configuration
public class RabbitMqConfiguration {
    @Value("${rabbitmq.uri}")
    private String rabbitUri;

    @Value("#{'${rabbitmq.queueNames}'.split(',')}")
    private String[] queueNames;

    @Value("${rabbitmq.recoveryInterval}")
    private Long recoveryInterval;

    @Value("${rabbitmq.receiveTimeout}")
    private Long receiveTimeout;

    @Bean
    SimpleMessageListenerContainer container(ConnectionFactory connectionFactory,
                                             RabbitMqConsumer messageListener,
                                             MessageErrorHandler errorHandler) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames(queueNames);
        container.setMessageListener(messageListener);
        container.setRecoveryInterval(recoveryInterval);
        container.setReceiveTimeout(receiveTimeout);
        container.setDefaultRequeueRejected(false);
        container.setErrorHandler(errorHandler);
        return container;
    }

    @Bean
    ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(URI.create(rabbitUri));
    }
}
```

其中，`DefaultRequeueRejected`如果被设置成`false`，表示当出现异常时不会将消息保留在当前Queue中，如果设置为`true`(默认值)，表示出错后会将消息保留在当前Queue中，并且应用程序会不停地读取消息，应根据实际需求处理，通常可以采用Dead Letter(死信)的方式处理，后续会详解。

消息监听者`RabbitMqConsumer`实现了`MessageListener`接口，消息会被发送到`onMessage`方法，子系统需要在这里处理接收到的消息。
``` java RabbitMqConsumer.java
package ws.message.consumer;
import static java.lang.String.format;
import static org.apache.log4j.Logger.getLogger;
import static org.joda.time.DateTime.now;
import org.apache.log4j.Logger;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;
import ws.message.repository.MessageRepository;

@Component
public class RabbitMqConsumer implements MessageListener {
    private final static Logger LOGGER = getLogger(RabbitMqConsumer.class);

    @Autowired
    private MessageRepository messageRepository;

    @Transactional
    @Override
    public void onMessage(Message message) {
        String bodyContent = new String(message.getBody());
        LOGGER.info(format("@@@@@@@@%s Received: %s\n", now(), bodyContent));
        messageRepository.save(new ws.message.entity.Message(bodyContent));
    }
}
```

由于子系统处理完成消息后会与数据库进行交互，这里也会配置内存数据库来模拟实际行为，同时，需要设置`uri`和`queueName`，这里的uri是连接RabbitMQ的固定格式，也可以采用源系统配置中分开的写法，另外，`recoveryInterval`表示Queue恢复重试的时间间隔，重试次数默认无限制，可以单独配置`FixedBackOff`，`receiveTimeout`接收消息超时时间，都以毫秒为单位。
``` yml application.yml
server:
  port: 8080
  context-path: /consumer

spring:
  datasource:
    url: jdbc:h2:./.tmp/msgdb
    username: sa
    password:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

rabbitmq:
  uri: amqp://guest:guest@192.168.56.105:5672
  queueNames: DEV.MESSAGE.QUEUE.APP1
  recoveryInterval: 10000
  receiveTimeout: 1000
```

完整代码可以参考GitHub Demo: [message-consumer](https://github.com/Waterstrong/message-consumer)。

##### 全局事务处理
既然这里涉及到消息机制和数据库的操作，必定需要考虑全局事务提交和回滚的情况，如果对事务还不太了解可以参阅之前的博客 [JTA实现分布式事务](/xa-transactions-with-jta) 和 [事务处理机制与协议](/transactional-mechanism-protocol)。在Spring Boot项目中，除了添加`spring-boot-starter-amqp`和`spring-boot-starter-data-jpa`依赖支持RabbitMQ消息和JPA数据库操作外，还需要添加`spring-boot-starter-jta-bitronix`依赖引入`Bitronix`支持全局事务机制，这样`DataSource`和`ConnectionFactory`会默认被加入到XA资源管理中。

当子系统收到消息处理后，在准备保存数据库时发生了异常，消息和数据库都会被回滚，如果配置了`DefaultRequeueRejected`为`false`，消息会被立即丢弃或转到其他Queue上，当然可以在丢弃之前记录下日志或进行异常处理，该值默认会为`true`，假设没有特殊配置，消息都会一直保留在当前Queue中，应用程序会一直不停读取消息，这样会阻塞后续的消息，因此必须设置消息在一定的重试次数后应被丢弃，一种常用的手段是为当前Queue配置`message-ttl`和`dead-letter-exchange`实现消息的超时和转发。
- message-ttl：一个消息在Queue上可以停留的时间，如果消息在规定时间内未消费将被视为超时过期并丢弃，时间单位为毫秒。
- dead-letter-exchange：可以为当前Queue配置某个Exchange或Queue，当消息被拒绝或过期时，消息会被转发到配置的Exchange上。

如果配置了`dead-letter-exchange`，那么可以设置在一定时间后再将消息以同样的形式返回到当前的Queue中，这样就实现了重试的机制，但该方法需要在消息头中记录重试的次数并用程序判断次数，以防止无限循环。

### 结束语
在处理消息的回滚和重试时，可能还需要再寻求一些其他更好的技术解决方案，并且需要保证在应用程序或RabbitMQ服务器突然挂掉重启后，能够再次读取并处理之前失败的消息，只有当超过了一定的时间或重试次数时才将消息丢弃并将副本存在硬盘日志文件中。


----
References

* [RabbitMQ Home](http://www.rabbitmq.com/)
* [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)
* [Messaging with RabbitMQ](https://spring.io/guides/gs/messaging-rabbitmq/)
* [事务处理机制与协议](http://blog.waterstrong.me/transactional-mechanism-protocol/)
* [JTA实现分布式事务](http://blog.waterstrong.me/xa-transactions-with-jta/)