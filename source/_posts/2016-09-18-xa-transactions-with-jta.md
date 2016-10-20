---
title: JTA实现分布式事务
date: 2016-09-18 22:05:32
category: Techniques
tags: [分布式事务, JTA, Bitronix, Spring Boot]
description: 如何在Spring中基于JTA实现分布式事务并实践？Spring Boot除了对非XA的事务进行了封装处理，并提供了注解Annotation的方式实现事务管理，也对多XA资源的分布式JTA事务提供了很好的支持，通常可选的内嵌事务管理器有Atomikos和Bitronix。
published: false
---

## 开门见山
关于事务的基本概念和介绍请参阅另一篇博客[事务处理机制与协议](transactional-mechanism-protocol)，这里着重讲解如何在Spring中基于JTA实现分布式事务并实践。

Spring Boot除了对非XA的事务进行了封装处理，并提供了注解Annotation的方式实现事务管理，也对多XA资源的分布式JTA事务提供了很好的支持，通常可选的内嵌事务管理器有`Atomikos`和`Bitronix`，任意选择一个作为实现XA事务管理的管理器即可。

接下来就分别介绍利用Spring Boot实现非XA和XA的事务处理。


## Spring对非XA事务的实现
使用Spring Boot实现非XA事务非常简单，只需要简单的两步即可：
1. 在构建脚本文件中加入依赖`org.springframework.boot:spring-boot-starter-jdbc`。
2. 无论采用基本的`JdbcTemplate`或封装的`JPA`与数据库进行交互，只需要在具体方法上加注解`@Transactional`。

这样就将该块的操作序列当作一个事务单元进行管理，任何失败或异常都将导致所有操作序列回滚，一个具体的Demo可参见[Managing Transactions](https://spring.io/guides/gs/managing-transactions/)，另外，如果注解加在类名上，则代表该类中每个方法都将作为一个事务单元被管理。


## 采用Atomikos事务管理器


## 采用Bitronix事务管理器
Bitronix是一款JTA事务管理器的开源库，可以通过添加依赖`spring-boot-starter-jta-bitronix`，然后Spring Boot会自动配置Bitronix，默认的事务日志文件会被写在项目下的`transaction-logs`文件目录下的`part1.btm`和`part2.btm`中，可以通过配置`spring.jta.log-dir`来指定日志目录。可以设置`spring.jta.bitronix.properties`读取相应的配置文件，相当于自定义配置`bitronix.tm.configuration`实例。

为了保证多个事务管理器能够正常安全地协调相同的资源管理器，每一个Bitronix实例必须配置一个唯一的ID，默认值为当前所在机器IP，通常在产品环境中，可以为每个应用程序实例配置`spring.jta.transaction-manager-id`为不同的值。

#### Transaction Manager Configuration


#### General Databases


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