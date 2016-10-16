---
title: AOP面向切面编程
date: 2016-01-20 13:45:32
category: Design Patterns
tags: [面向切面编程, AOP, 反射机制, 代理模式, Proxy, Java, CGLib]
description: AOP(Aspect Oriented Programming)，即面向切面编程，是一种设计思想，通过预编译方式或运行时动态代理实现为程序统一添加功能的技术。
---

## What is AOP ?
### 基本介绍

AOP(Aspect Oriented Programming)，即面向切面编程，是一种设计思想，通过预编译方式或运行时动态代理实现为程序统一添加功能的技术。

AOP可以理解为OOP(面向对象编程)里程碑式的补充。OOP是从静态角度考虑程序结构，从横向上区分出类，AOP是从动态角度考虑程序运行过程，从纵向上向对象中加入特定的代码，加上时间维度，AOP使得OOP从二维变成三维，由平面变成立体。常常通过AOP来处理一些具有横切性质的系统性服务，如[事务处理](/transactional-mechanism-protocol)、安全检查、缓存、对象池管理等。

通常做法，是在运行时动态地将代码切入到类的指定方法、指定位置上，而切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点，采用AOP可以把几个类共有的代码抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。接下来将详解AOP中的几个核心概念。

### AOP相关概念
#### Aspect（切面/方面）
需要切入到指定类指定方法的代码片段称为切面，包含定义的额外功能的某个模块，相当于OOP中定义的类，但代表的更多是对象间横向的关系，如日志切面、权限切面、事务切面等。

#### Joinpoint（连接点）
程序执行过程中精确执行点，如类中的方法调用、异常抛出等。

#### Advice（通知/增强）
在切面的某个特定的连接点(Joinpoint)上执行的动作，是执行切面的具体逻辑，包括Before，AfterReturning，AfterThrowing，After，Around等Advice。
   - Before Advice: 前置通知，在方法执行前执行
   - AfterReturning: 后置返回通知，在方法执行正常返回后执行，方法没有抛出任何异常
   - AfterThrowing Advice: 后置异常通知，在方法执行过程中抛出异常的时候执行
   - After Advice: 后置通知/后置最终通知，在方法执行完成后执行，不论是正常返回还是异常退出
   - Around Advice: 环绕通知，在方法执行前后和抛出异常时执行，环绕通知可以在方法调用前后完成自定义的行为

#### Pointcut（切入点）
匹配连接点(Joinpoint)的断言，通知(Advice)和切入点(Pointcut)表达式关联，并在满足该切入点(Pointcut)的连接点(Joinpoint)上运行，可以认为是连接点的集合，本质是一个捕获连接点的结构，通常可用正则表达式来找到那些匹配的连接点。

#### Target（目标对象）
包含连接点的对象，即被通知或被代理的实际对象。

#### Proxy（代理对象）
通过代理模式创建的对象，从而实现在目标对象连接点处插入切面，代理对象对使用者透明，是程序运行中间产物。

#### Weaving（织入）
织入是一个过程，将切面应用到目标对象从而创建出AOP代理对象的过程，织入可以在编译期(Complie)、类装载期(Classload)、运行期(Runtime)进行。

----

## How does it wok ?
AOP技术可以通过反射机制与动态代理机制实现，业务逻辑组件在运行过程中，动态创建一个代理对象供使用者调用，该代理对象已经将切面成功切入到目标方法的连接点上，从而使切面的功能与业务逻辑的功能同时得以执行。当然也可以通过静态织入的方式，引入特定的语法，在编译期间织入有关切面代码。

反射机制被视为动态（或准动态）语言的一个关键性质，它允许取得任何一个已知名称的class的内部信息，并能直接操作程序的内部属性。
动态代理机制指在程序运行时运用反射机制动态创建代理类，后续会有举例讲解到。

从网络图库中收割到一幅关于AOP理解的图例，表达得非常到位，可以参见一下:
![](/assets/aop/aop_diagram.jpg)
图中的切入箭头代表切入的动作，而切入的动作是依赖于称为切入点(Pointcut)的表达式规则匹配找到应该切入到哪些连接点(Joinpoint)上。从图解中可以大致了解到AOP的工作原理和过程，同时加强了对切面、连接点和切入点区别和关系的进一步理解。

----

## Why AOP ?
OOP引入封装、继承和多态来建立对象层次结构，从而模拟公共行为的一个集合，当需要为分散的对象引入公共行为的时候，OOP则显得无能为力，如日志、权限、事务等功能。这种与主要业务处理流程（核心关注点）关系不大的代码被称为横切（cross cutting），在OO设计中会导致大量重复代码，也不利于模块重用和维护。

因此需要AOP作为OOP的补充和完善来解决上述问题，AOP的优势主要表现在以下方面：
1. 解决核心业务逻辑共有代码冗余问题，使代码变更简洁优雅
2. 只需关心核心的业务逻辑处理，能够提高编写工作效率
3. 核心关注点和横切关注点分离开来，共有代码集中存放，使得维护工作更加简单轻松

----

## When AOP?
其实之前多次提到，当需要为分散的对象引入公共行为的时候，需要处理一些具有横切性质的系统性服务的时候，就可以使用AOP，以下列举了常见的几种应用范围和场合:
- Authentication 权限检查
- logging / tracing 日志/跟踪
- Transactions 事务处理
- Exception Handling 异常处理
- profiling / monitoring　分析/监控
- Caching 缓存
- Debugging 调试
- Performance Optimization 性能优化
- Persistence 持久化
- Resource Pooling 资源池
- Synchronization 同步
- ……

----

## AOP in Practice
主要针对AOP在Java中的实现，并解读在Spring中AOP的伪代码实现。

### 代理设计模式
首先来回顾一下设计模式中的代理模式:
> 代理模式(Proxy Design Pattern)，为其他对象提供一种代理以控制对这个对象的访问。

也就是如下图所示的关系，代理对象通过调用实际对象的方法来控制对该实际对象的访问。

![](/assets/aop/proxy_dp.png)

以下简单给出以Hello类为例的伪代码，通过代理对象调用实际对象的sayHello方法，并且在该方法的实际调用前后增加打印记录行为，模拟了拦截方法的过程。

``` java
public interface IHello {
    void sayHello(String name);
}

// 实际的类，已有的操作和行为
public class Hello implements IHello {
    @Override
    public void sayHello(String name) {
         println("Hello "+name);
    }
}

// 代理类，通过调用代理方法访问实际方法并且添加新的职责
class HelloProxy implements IHello {
    private IHello hello = new Hello(); // 指向目标对象,通常在用构造方法传值

    @Override
    public void sayHello(String name) {
        println("before...."); // 实际调用前添加新的行为
        hello.sayHello(name);
        println("after...."); // 实际调用后添加新的行为
    }
}
```

在回顾了代理模式后，那么进一步了解如何结合代理模式在运行时通过反射机制动态创建代理类，即动态代理技术。

### JDK动态代理
主要针对JDK动态代理方式，使用接口实现，给出核心部分伪代码：

``` java
public class DynamicProxyHello implements InvocationHandler {
    private Object target; // 目标对象
    public Object bind(Object target){
        this.target = target;
        // 通过目标类和接口来生成代理类
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                                      target.getClass().getInterfaces(), this);
    }

    @Override // 代理类中拦截了目标对象的方法
    public Object invoke(Object proxy, Method method, Object[] args) {
        do("before..... "); // 代理类中Advice
        Object result = method.invoke(target, args); // 目标对象的实际方法
        do("after.....");
        return result;
    }
}
```

特别注意: JDK动态代理只能基于接口动态代理。一般默认情况下，如果目标类是接口，则使用JDK动态代理技术，否则只能使用CGLib来生成代理。

### CGLib代理
CGLIB方式，使用继承实现：

``` java
// 主要了解以下几个类, 代码在相应的库中都可以找到, 此处不再赘述

MethodInterceptor // 方法拦截类, 定义的代理类需要实现该接口以调用intercept方法添加Advice

Enhancer // 增强类, 继承至AbstractClassGenerator, 主要用于生成目标类的子类

MethodProxy // 生成的子类, 可以在intercept中通过调用proxy.invokeSuper()来调用目标对象的实际方法
```

特别注意: CGLib代理基于接口和非final类代理，不能代理static方法。

之前有用Java实现过简单的IoC和AOP，有兴趣可以参见工程: [summarine](https://github.com/Waterstrong/summarine)。