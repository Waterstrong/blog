---
title: 使用Stubby4j
date: 2016-09-05 22:29:26
category: Tools
tags: [Stub, Mock, HTTP, REST]
description: Stubby4J一款灵活可配置的基于HTTP(s)协议测试Web服务交互的工具，采用内嵌式的Jetty作为HTTP服务器，它的主要作用在于，可以在集成测试时，用来模拟第三方Web服务的API行为，比如，目前比较流行的RESTful架构风格的Web服务，
published: true
---

## 什么是stubby4j？
Stubby4J项目是由个人发起的开源项目，它是一款非常灵活可配置的基于HTTP(s)协议测试Web服务交互的工具，采用内嵌式的Jetty作为HTTP服务器，它的主要作用在于，可以在集成测试时，用来模拟第三方Web服务的API行为，比如，目前比较流行的RESTful架构风格的Web服务。

Stubby4J是由Java编写，目前使用一些第三依赖有：
- javax.servlet-api-3.1.0.jar
- jetty-server-9.2.10.v20150310.jar
- jetty-servlets-9.2.10.v20150310.jar
- jetty-http-9.2.10.v20150310.jar
- jetty-io-9.2.10.v20150310.jar
- jetty-continuation-9.2.10.v20150310.jar
- jetty-util-9.2.10.v20150310.jar
- commons-cli-1.2.jar
- snakeyaml-1.15.jar
- jsonassert-1.2.3.jar
- xmlunit-1.6.jar
- json-20090211.jar

## 为什么使用Stubby4J？
- 模拟HTTP请求，Stub第三方API的返回数据
- 在写集成测试时，Mock第三方API更加便捷
- 能够验证发送的所有参数并指定详细返回数据
- 目前支持所有HTTP方法：GET, POST, PUT, PATCH, DELETE, HEAD等
- 支持HTTP和HTTPS协议，同时可模拟返回的错误码
- 在性能测试和稳定性测试时，支持定义延时返回
- 使用相对简单，配置非常便捷，启动也很快速

## 如何使用Stubby4J？
#### Gradle中配置依赖
在`build.gradle`中加入其依赖：
```
compile 'by.stub:stubby4j:3.3.0'
```

#### 启动与停止stubby4j
在集成测试启动之前执行`startJetty`，常用启动方法如下:
```
startJetty("stubby4j.yml")  # localhost默认端口: Stubs(8882), Admin(8889) and SslStubs portals(7443) 

startJetty(8882, "stubby4j.yml") # 可以指定Stubs端口，其它为默认值

startJetty(8882, 8889， "stubby4j.yml") # 可以指定Stubs和Admin端口，其它为默认值
```

在集成测试运行完成后需要关闭stubby4j服务：
```
stopJetty()
```

#### 基于YAML文件的示例
示例一：模拟GET请求并返回Json格式Payload
```
-  request:
      method: GET
      url: /users/111
   response:
      status: 200
      headers:
         content-type: application/json
      body: >
      	{"userId": 111, "userName": "Peter"}
```

示例二：在request时指定多个methods
```
-  request:
      url: /anything
      method: [GET, HEAD]

-  request:
      url: /anything
      method:
         -  GET
         -  HEAD
```

示例三：可以指定查询参数
```

```

示例四：POST时指定发送的Payload
```

```

示例五：返回的Response是Json文件
```

```

更多示例可以参考[github stubby4j](https://github.com/azagniotov/stubby4j)。

----
References
* [github stubby4j](https://github.com/azagniotov/stubby4j)