---
title: 事务处理机制与协议
date: 2016-04-04 19:11:03
category: Transaction
tags: [事务, 数据库, ACID特性, XA协议, 分布式事务, 锁机制, 行版本控制]
---

## 事务定义及其特性
### 什么是事务？
> A transaction is a unit of work that you want to treat as "a whole". It has to either happen in full, or not at all.

直接地讲，就是事务是一个整体，其中的若干处理要么都做，要么都不做。接下来我们就详细地聊聊事务。

<!-- more -->

### 事务的ACID特性
事务的四大特性分别为原子性、一致性、隔离性和永久性，称为ACID特性，也称酸性。

- 原子性（Atomicity）
  + 一个事务中所有操作是一个不可分割的操作序列，要么全做，要么全不做
- 一致性（Consistency）
  + 数据不会因为事务的执行而遭到破坏，一致性保证了这个事务所包含的一系列的操作完成后系统仍然在一个一致的状态，侧重点在于，事务执行前后在某种程度上是等价的，从一个一致状态到另一个一致状态。例如，对银行转帐事务，不管事务成功还是失败，应该保证事务结束后两人存款总额为定值
- 隔离性（Isolation）
  + 一个事务的执行，不受其他事务（进程）的干扰，既并发执行的个事务之间互不干扰，每个事务都有各自的完整数据空间
- 永久性（Durability）
  + 一个事务一旦提交，它对数据库所作的改变将是永久的，使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态

### 数据库事务管理
对于数据库来说，事务的ACID特性是由关系数据库管理系统（RDBMS，数据库系统）来实现的。

1. DBMS采用**日志**来保证事务的原子性、一致性和持久性。日志记录了事务对数据库所做的更新，如果某个事务在执行过程中发生错误，就可以根据日志撤销事务对数据库已做的更新，使数据库退回到执行事务前的初始状态。

2. DBMS采用**锁机制**来实现事务的隔离性。每个事务对所依赖的资源（如行、页或表）请求不同类型的锁，当多个事务同时更新数据库中相同的数据时，只允许持有锁的事务能更新该数据，其他事务必须等待，直到前一个事务释放了锁，其他事务才有机会更新该数据。锁可以阻止其他事务以某种可能会导致事务请求锁出错的方式修改资源，当事务不再依赖锁定的资源时，它将释放锁。通常会有共享(S)锁、排它(X)锁、更新(U)锁等，各锁的解释、并发效率、锁冲突及其防止办法不在本文范围，有兴趣可以自行了解。

3. 除了锁定外，也可能采用**行版本控制**来实现事务隔离性。当启用了基于行版本控制的隔离级别时，数据库引擎将维护修改的每一行的版本，应用程序可以指定事务使用行版本查看事务或查询开始时存在的数据，而不是使用锁保护所有读取。通过使用行版本控制，读取操作阻止其他事务的可能性将大大降低，锁数量减少，死锁可能性降低，有效减少了管理锁的开销。

锁定和行版本控制可以防止用户读取未提交的数据，还可以防止多个用户尝试同时更改同一数据。如果不进行锁定或行版本控制，对数据执行的查询可能会返回数据库中尚未提交的数据，从而产生意外的结果。

在了解数据库保证事务ACID特性的基本原理后，那么……对程序提供的数据库编程接口，如何实现事务的呢？接下来介绍本地事务。

----

## 本地事务（Local Transaction）
本地事务(Local Transaction)主要指限制在单个进程内的事务，不涉及多个数据库源，通常会有Begin Transaction … End Transaction来控制事务的开始与结束。以对数据库访问为例，接下来用伪代码实现事务的提交/回滚。

### 本地事务 模型1
设置自动提交的情况：

```
conn = getConnection(“url","user","pwd");
conn.setAutoCommit(true);
try{
    conn.execute(...);
}
catch(Exception e) {
    print(e);
}
finally {
    conn.close();
}
```
### 本地事务 模型2
设置非自动提交情况：

```
conn = getConnection(“url","user","pwd");
conn.setAutoCommit(false);
try{
    conn.execute(…);
    conn.commit();
}
catch(Exception e) {
    conn.rollback();
}
finally {
    conn.close();
}
```
这也是最早的访问方式，相信你也发现一些问题了，如果设置为自动提交，当某个操作需要多个执行序列完成时，那么每次execute时都会commit，在很程度上降低了执行效率。若设置为非自动提交，虽然解决了多次execute的问题，但直接暴露conn并不是理想的做法，同时还需要手动close连接。因此，有没有更好地方式呢？答案是肯定的，接下来就看看编程式事务的实现方式。

----

## 编程式事务（Programmatic Transaction）

编程式事务(Programmatic Transaction)通过编程语言提供的事务API和事务服务提供者进行事务控制。通常的做法是在代码中直接加入处理事务的逻辑，显式地调用其commit()、rollback()等事务管理相关方法。

### 编程式事务 模型
```
userTransaction.begin();
try {
    doSomething();
    doAnotherThing();
    userTransaction.commit();
} catch (Exception ex) {
    userTransaction.rollback();
}
```

Programmatic与Local Transaction的区别在于Programmatic把Local方式下的conn封装起来，并且手动控制commit和rollback，在一定程序上简化了编程的繁琐性，更加关注事务开始、提交与回滚。你觉得这种方式就已经很棒了么，再想想还有没有更好更创新的方式呢，声明式事务会给你想要的答案。

----

## 声明式事务（Declarative Transaction）

声明式事务(Declarative Transaction)对目标方法上添加注解(Annotation)或在配置文件中定义，通过对方法前后拦截添加事务处理逻辑。虽然XML配置的方式在前几年很受欢迎，也是具有里程碑的意义，但小编我更青睐注解的方式，况且目前主流的IoC框架也都支持注解方式并且推荐使用。接下来将给出Java形式的伪代码进行解释。

### 声明式事务 模型
```
@Transaction
void doSomething() {
    process1();
    process2();
    ……
    repository.save();
}
```
其中的@Transaction就是一个注解(Annotation)，其内部实现原理通常采用的是AOP(面向切面编程)的方式进行方法的拦截。
```
Object intercept(proxy, method, args) {
    trans.begin();
    try {
    	method.invoke(target, args);
    	trans.commit();
    } catch (Exception e) {
    	trans.rollback();
    }
}
```

以上所有的事务实现方式都主要集中在单一数据库情况，那么对于多数据库协调或者混合数据源情形，如数据库加消息队列等，又如何保证事务正确有效地执行呢？答案是分布式事务，也称全局事务来管理了。

----

## 全局/分布式事务（Global/Distributed Transaction）
跨越多个数据库或进程，多资源协调（例如：访问多个数据库，或数据库加消息队列，又或是多个消息队列等）
事务管理器（Transaction Manager）
资源管理器（Resource Manager）
XA Standard / Two-Phase Commit

### Merge/Split实例

Eg: Identity Mapping API Merge/Split
JMS Message触发数据库更新操作

#### 伪代码(DB+JMS)

#### 成功调用过程

#### 可能失败的情形

### Global Transaction 如何协调多资源？

多个数据库？
多个消息队列/主题？
……


X/Open XA规范接口


XA事务实现原理


XA两阶段提交协议

谁是卧底？

TM/RM Init 伪代码


XA 2PC 伪代码


## 结束语 Last but Not the Last

事务的实现方式与分类
对于Local事务，范围仅限于单个可识别数据资源的事务操作。
对于Global事务，可能出现timeout问题，连锁反应导致系统变慢，同时会消耗更多性能资源。
仅在同一个事务上下文中需要协调多种资源时，才有必要使用XA。也就是说仅当多个资源必须在同一个事务范畴内被协调时，才有必要用XA。


## 思考和深入学习

