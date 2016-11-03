---
title: RabbitMQ项目实践应用 - 发布订阅模式
date: 2016-11-01 23:26:31
category: Frameworks
tags: [AMQP, RabbitMQ, Message, Topic, Queue]
thumbnailImage: /assets/rabbitmq-guide/rabbitmq_logo.png
description: 利用RabbitMQ解决项目中实际需求问题
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
##### 配置RabbitMQ服务器


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

在默认的`resources`目录下创建`application.yml`并添加需要的Properties，需要指定`host`、`username`、`password`以及`topicName`等配置参数，另外，`recoveryInterval`表示恢复重试的时间间隔，`connectionTimeout`表示连接超时，都以毫秒为单位。
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
                                             RabbitMqConsumer messageListener) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames(queueNames);
        container.setMessageListener(messageListener);
        container.setRecoveryInterval(recoveryInterval);
        container.setReceiveTimeout(receiveTimeout);
        container.setDefaultRequeueRejected(false);
        return container;
    }

    @Bean
    ConnectionFactory connectionFactory() {
        return new CachingConnectionFactory(URI.create(rabbitUri));
    }
}
```

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

由于子系统处理完成消息后会与数据库进行交互，这里也会配置内存数据库来模拟实际行为，同时，需要设置`uri`和`queueName`，这里的uri是连接RabbitMQ的固定格式，也可以采用源系统配置中分开的写法，另外，`recoveryInterval`表示恢复重试的时间间隔，`receiveTimeout`接收消息超时时间，都以毫秒为单位。
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


##### 消息DXL

### 结束语

In production


----
References

* [RabbitMQ Home](http://www.rabbitmq.com/)
* [RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)