---
title: JTA实现分布式事务
date: 2016-09-18 22:05:32
category: Techniques
tags: [分布式事务, JTA, Bitronix, Spring Boot]
description: distributed-transactions-with-jta
published: false
---

## 开门见山
关于事务的基本概念和介绍请参阅另一篇博客[事务处理机制与协议](transactional-mechanism-protocol)，这里着重讲解如何在Spring中基于JTA实现分布式事务并进行代码实践。

## 采用Atomikos事务管理器


## 采用Bitronix事务管理器


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