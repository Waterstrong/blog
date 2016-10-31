---
title: JTA实现分布式事务
date: 2016-10-18 22:05:32
category: Techniques
tags: [分布式事务, JTA, Bitronix, Spring Boot]
thumbnailImage: /assets/common-pic/vintage1.jpg
description: Spring Boot除了对非XA的事务进行了封装处理，并提供了注解@Transactional的方式实现事务管理，也对多XA资源的分布式JTA事务提供了很好的支持，通常可选的内嵌事务管理器有Atomikos和Bitronix。
published: true
---

## 开门见山
关于事务的基本概念和介绍请参阅另一篇博客[事务处理机制与协议](/transactional-mechanism-protocol)，这里着重讲解如何在Spring中基于JTA(Java Transaction API)实现分布式事务并实践。

Spring Boot除了对非XA的事务进行了封装处理，并提供了注解@Transactional的方式实现事务管理，也对多XA资源的分布式JTA事务提供了很好的支持，通常可选的内嵌事务管理器有`Atomikos`和`Bitronix`，任意选择一个作为实现XA事务管理的管理器即可。

接下来就分别介绍利用Spring Boot实现非XA和XA的事务处理。


## Spring对非XA事务的实现
使用Spring Boot实现**非XA事务**非常简单，只需要简单的两步即可：
1. 在构建脚本文件中加入依赖`org.springframework.boot:spring-boot-starter-jdbc`。
2. 无论采用基本的`JdbcTemplate`或封装的`JPA`与数据库进行交互，只需要在具体方法上加注解`@Transactional(org.springframework.transaction.annotation.Transactional)`。

这样就将该块的操作序列当作一个事务单元进行管理，任何失败或异常都将导致所有操作序列回滚，一个具体的简单Demo可参见[Transaction Demo](https://github.com/Waterstrong/spring-bitronix/tree/master/src/main/java/ws/transaction/demo)，另外，如果注解加在类名上，则代表该类中每个方法都将作为一个事务单元被管理。


## 采用Atomikos事务管理器


## 采用Bitronix事务管理器
Bitronix是一款JTA事务管理器的开源库，可以通过添加依赖`spring-boot-starter-jta-bitronix`，然后Spring Boot会自动配置Bitronix，默认的事务日志文件会被写在项目下的`transaction-logs`文件目录下的`part1.btm`和`part2.btm`中，可以通过配置`spring.jta.log-dir`来指定日志目录。可以设置`spring.jta.bitronix.properties`读取相应的配置文件，相当于自定义配置`bitronix.tm.configuration`实例。

为了保证多个事务管理器能够正常安全地协调相同的资源管理器，每一个Bitronix实例必须配置一个唯一的ID，默认值为当前所在机器IP，通常在产品环境中，可以为每个应用程序实例配置`spring.jta.transaction-manager-id`为不同的值。

一个具体的Bitronix分布式事务管理示例可参见[Bitronix Demo](https://github.com/Waterstrong/spring-bitronix/tree/master/src/main/java/ws/xa/bitronix/demo)。

#### Transaction Manager Configuration
可以根据自定义需求在应用程序配置文件中配置以下参数：
``` bash
spring.jta.bitronix.connectionfactory.acquire-increment=1 # Number of connections to create when growing the pool.
spring.jta.bitronix.connectionfactory.acquisition-interval=1 # Time, in seconds, to wait before trying to acquire a connection again after an invalid connection was acquired.
spring.jta.bitronix.connectionfactory.acquisition-timeout=30 # Timeout, in seconds, for acquiring connections from the pool.
spring.jta.bitronix.connectionfactory.allow-local-transactions=true # Whether or not the transaction manager should allow mixing XA and non-XA transactions.
spring.jta.bitronix.connectionfactory.apply-transaction-timeout=false # Whether or not the transaction timeout should be set on the XAResource when it is enlisted.
spring.jta.bitronix.connectionfactory.automatic-enlisting-enabled=true # Whether or not resources should be enlisted and delisted automatically.
spring.jta.bitronix.connectionfactory.cache-producers-consumers=true # Whether or not produces and consumers should be cached.
spring.jta.bitronix.connectionfactory.defer-connection-release=true # Whether or not the provider can run many transactions on the same connection and supports transaction interleaving.
spring.jta.bitronix.connectionfactory.ignore-recovery-failures=false # Whether or not recovery failures should be ignored.
spring.jta.bitronix.connectionfactory.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool.
spring.jta.bitronix.connectionfactory.max-pool-size=10 # The maximum size of the pool. 0 denotes no limit.
spring.jta.bitronix.connectionfactory.min-pool-size=0 # The minimum size of the pool.
spring.jta.bitronix.connectionfactory.password= # The password to use to connect to the JMS provider.
spring.jta.bitronix.connectionfactory.share-transaction-connections=false #  Whether or not connections in the ACCESSIBLE state can be shared within the context of a transaction.
spring.jta.bitronix.connectionfactory.test-connections=true # Whether or not connections should be tested when acquired from the pool.
spring.jta.bitronix.connectionfactory.two-pc-ordering-position=1 # The position that this resource should take during two-phase commit (always first is Integer.MIN_VALUE, always last is Integer.MAX_VALUE).
spring.jta.bitronix.connectionfactory.unique-name=jmsConnectionFactory # The unique name used to identify the resource during recovery.
spring.jta.bitronix.connectionfactory.use-tm-join=true Whether or not TMJOIN should be used when starting XAResources.
spring.jta.bitronix.connectionfactory.user= # The user to use to connect to the JMS provider.
spring.jta.bitronix.datasource.acquire-increment=1 # Number of connections to create when growing the pool.
spring.jta.bitronix.datasource.acquisition-interval=1 # Time, in seconds, to wait before trying to acquire a connection again after an invalid connection was acquired.
spring.jta.bitronix.datasource.acquisition-timeout=30 # Timeout, in seconds, for acquiring connections from the pool.
spring.jta.bitronix.datasource.allow-local-transactions=true # Whether or not the transaction manager should allow mixing XA and non-XA transactions.
spring.jta.bitronix.datasource.apply-transaction-timeout=false # Whether or not the transaction timeout should be set on the XAResource when it is enlisted.
spring.jta.bitronix.datasource.automatic-enlisting-enabled=true # Whether or not resources should be enlisted and delisted automatically.
spring.jta.bitronix.datasource.cursor-holdability= # The default cursor holdability for connections.
spring.jta.bitronix.datasource.defer-connection-release=true # Whether or not the database can run many transactions on the same connection and supports transaction interleaving.
spring.jta.bitronix.datasource.enable-jdbc4-connection-test= # Whether or not Connection.isValid() is called when acquiring a connection from the pool.
spring.jta.bitronix.datasource.ignore-recovery-failures=false # Whether or not recovery failures should be ignored.
spring.jta.bitronix.datasource.isolation-level= # The default isolation level for connections.
spring.jta.bitronix.datasource.local-auto-commit= # The default auto-commit mode for local transactions.
spring.jta.bitronix.datasource.login-timeout= # Timeout, in seconds, for establishing a database connection.
spring.jta.bitronix.datasource.max-idle-time=60 # The time, in seconds, after which connections are cleaned up from the pool.
spring.jta.bitronix.datasource.max-pool-size=10 # The maximum size of the pool. 0 denotes no limit.
spring.jta.bitronix.datasource.min-pool-size=0 # The minimum size of the pool.
spring.jta.bitronix.datasource.prepared-statement-cache-size=0 # The target size of the prepared statement cache. 0 disables the cache.
spring.jta.bitronix.datasource.share-transaction-connections=false #  Whether or not connections in the ACCESSIBLE state can be shared within the context of a transaction.
spring.jta.bitronix.datasource.test-query= # SQL query or statement used to validate a connection before returning it.
spring.jta.bitronix.datasource.two-pc-ordering-position=1 # The position that this resource should take during two-phase commit (always first is Integer.MIN_VALUE, always last is Integer.MAX_VALUE).
spring.jta.bitronix.datasource.unique-name=dataSource # The unique name used to identify the resource during recovery.
spring.jta.bitronix.datasource.use-tm-join=true Whether or not TMJOIN should be used when starting XAResources.
spring.jta.bitronix.properties.allow-multiple-lrc=false # Allow multiple LRC resources to be enlisted into the same transaction.
spring.jta.bitronix.properties.asynchronous2-pc=false # Enable asynchronously execution of two phase commit.
spring.jta.bitronix.properties.background-recovery-interval-seconds=60 # Interval in seconds at which to run the recovery process in the background.
spring.jta.bitronix.properties.current-node-only-recovery=true # Recover only the current node.
spring.jta.bitronix.properties.debug-zero-resource-transaction=false # Log the creation and commit call stacks of transactions executed without a single enlisted resource.
spring.jta.bitronix.properties.default-transaction-timeout=60 # Default transaction timeout in seconds.
spring.jta.bitronix.properties.disable-jmx=false # Enable JMX support.
spring.jta.bitronix.properties.exception-analyzer= # Set the fully qualified name of the exception analyzer implementation to use.
spring.jta.bitronix.properties.filter-log-status=false # Enable filtering of logs so that only mandatory logs are written.
spring.jta.bitronix.properties.force-batching-enabled=true #  Set if disk forces are batched.
spring.jta.bitronix.properties.forced-write-enabled=true # Set if logs are forced to disk.
spring.jta.bitronix.properties.graceful-shutdown-interval=60 # Maximum amount of seconds the TM will wait for transactions to get done before aborting them at shutdown time.
spring.jta.bitronix.properties.jndi-transaction-synchronization-registry-name= # JNDI name of the TransactionSynchronizationRegistry.
spring.jta.bitronix.properties.jndi-user-transaction-name= # JNDI name of the UserTransaction.
spring.jta.bitronix.properties.journal=disk # Name of the journal. Can be 'disk', 'null' or a class name.
spring.jta.bitronix.properties.log-part1-filename=btm1.tlog # Name of the first fragment of the journal.
spring.jta.bitronix.properties.log-part2-filename=btm2.tlog # Name of the second fragment of the journal.
spring.jta.bitronix.properties.max-log-size-in-mb=2 # Maximum size in megabytes of the journal fragments.
spring.jta.bitronix.properties.resource-configuration-filename= # ResourceLoader configuration file name.
spring.jta.bitronix.properties.server-id= # ASCII ID that must uniquely identify this TM instance. Default to the machine's IP address.
spring.jta.bitronix.properties.skip-corrupted-logs=false # Skip corrupted transactions log entries.
spring.jta.bitronix.properties.warn-about-zero-resource-transaction=true # Log a warning for transactions executed without a single
```

#### General Databases
针对不同的数据库可能需要设置一些不同的参数开启XA功能，比如Postgresql，需要设置参数`max_prepared_transactions`，整型值，它决定能够同时处于prepared状态的事务的最大数目，0表示关闭prepared事务的特性，该值通常应该和max_connections的值一样大。
``` apacheconf postgresql.conf
max_prepared_transactions=30
```

如果使用AWS的Postgresql RDS数据库，则需要在Parameter Group中设置该值，否则默认会为0，表示关闭XA功能。

## 混合XA/非XA事务的JMS连接


----
References

* [Spring Distributed Transactions with JTA](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-jta.html)
* [Atomikos Home](https://www.atomikos.com/)
* [Class AtomikosProperties](http://docs.spring.io/spring-boot/docs/1.3.3.RELEASE/api/org/springframework/boot/jta/atomikos/AtomikosProperties.html)
* [Home of Bitronix JTA Transaction Manager](https://github.com/bitronix/btm)
* [Spring Boot JTA Code on GitHub](https://github.com/spring-projects/spring-boot/tree/v1.3.3.RELEASE/spring-boot/src/main/java/org/springframework/boot/jta)
* [Managing Transactions](https://spring.io/guides/gs/managing-transactions/)
* [Using Transactions](http://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html#commit_transactions)
* [Common application properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)