---
title: 在集成测试中使用Stubby4J
date: 2016-09-05 22:29:26
category: Tools
tags: [Stub, Mock, HTTP, 集成测试]
thumbnailImage: /assets/common-pic/car10.jpg
description: Stubby4J一款灵活可配置的基于HTTP(s)协议测试Web服务交互的工具，采用内嵌式的Jetty作为HTTP服务器，它的主要作用在于，可以在集成测试时，用来模拟第三方Web服务的API行为，比如，目前比较流行的RESTful架构风格的Web服务。
published: true
---

## 什么是stubby4j？
Stubby4J是基于Java编写的，该项目是由个人发起的开源项目，它是一款非常灵活可配置的基于HTTP(s)协议测试Web服务交互的工具，采用内嵌式的Jetty作为HTTP服务器，它的主要作用在于，可以在集成测试时，用来模拟第三方Web服务的API行为，比如，目前比较流行的RESTful架构风格的Web服务。

## 为什么使用Stubby4J？
- 模拟HTTP请求，Stub第三方API的返回数据
- 在写集成测试时，Mock第三方API更加便捷
- 能够验证发送的所有参数并指定详细返回数据
- 目前支持所有HTTP方法：GET, POST, PUT, PATCH, DELETE, HEAD等
- 支持HTTP和HTTPS协议，同时可模拟返回的错误码
- 在性能测试和稳定性测试时，支持定义延时返回
- 使用相对简单，配置非常便捷，启动也很快速

## 如何使用Stubby4J？
#### 命令行快速启动
首先需要[下载JAR包](http://search.maven.org/remotecontent?filepath=by/stub/stubby4j/3.3.0/stubby4j-3.3.0.jar)，假设本地已经安装了Java，然后在本地创建一个名为`cfg.yml`的文件：
``` yml cfg.yml
- request:
    method: GET
    url: /hello-world
  response:
    status: 200
    headers:
        Content-Type: application/json
    body: Hello World!
```

运行如下命令启动Stubby4J: 
``` bash
$ java -jar stubby4j-3.3.0.jar -d cfg.yml

Loaded: [GET] /hello-world
Admin portal configured at http://localhost:8889
Admin portal status enabled at http://localhost:8889/status
Stubs portal configured at http://localhost:8882
Stubs portal configured with TLS at https://localhost:7443 using internal keystore
Jetty successfully started
```

然后访问[http://localhost:8882/hello-world](http://localhost:8882/hello-world)可以看到返回的`Hello World!`字样。

如果需要进入Admin Portal，可以访问[http://localhost:8889/status](http://localhost:8889/status)查看Stub的数据。

#### 集成测试中的具体应用
首先在Gradle中配置依赖，在`build.gradle`中加入其依赖，只会在集成测试时使用，所以只需要加入`testCompile`即可：
``` gradle
testCompile 'by.stub:stubby4j:3.3.0'
```

然后需要在加载应用程序Context前启动Stubby4J，常用启动stubby4j的方法如下:
``` java
startJetty("stubby4j.yml")  # localhost默认端口: Stubs(8882), Admin(8889) and SslStubs portals(7443) 

startJetty(8882, "stubby4j.yml") # 可以指定Stubs端口，其它为默认值

startJetty(8882, 8889, "stubby4j.yml") # 可以指定Stubs和Admin端口，其它为默认值
```

在集成测试启动之前执行`startJetty`，需要在其Base父类中加入以下代码：
``` java
private static final StubbyClient API_STUB = new StubbyClient();

@BeforeClass
public static void startUp() throws Exception {
	API_STUB.startJetty(8882, new ClassPathResource("api/stubby4j.yml").getFile().getAbsolutePath());
}
```

其中`api/stubby4j.yml`文件位于集成测试代码的`resources`目录下。

在集成测试运行完成后需要停止stubby4j服务`stopJetty`：
``` java
@AfterClass
public static void shutDown() throws Exception {
	API_STUB.stopJetty();
}
```

具体示例可参阅GitHub Demo: [service-stubmock stubby4j](https://github.com/Waterstrong/service-stubmock/tree/master/src/main/java/ws/stubby4j/demo)。

#### 基于YAML文件的示例
**示例一：模拟GET请求并返回Json格式Payload**
``` yml
- request:
    method: GET
    url: ^/users/111$
  response:
    status: 200
    headers:
        content-type: application/json
    body: >
        {"userId": 111, "userName": "Peter"}

- request:
    method: GET
    url: /users/123
  response:
    status: 404
```

其中的`request:url`支持正则表达式，比如`^/[a-z]{3}-[a-z]{3}/[0-9]{2}/[A-Z]{2}/[a-z0-9]+$`。

**示例二：在request时指定多个methods**
``` yml
- request:
    url: /anything
    method: [GET, HEAD]

- request:
    url: /anything
    method:
        - GET
        - HEAD
```

**示例三：可以指定查询参数**
``` yml
- request:
    url: ^/with/parameters$
    method: GET
    query:
        search: search terms
        filter: month
```

其中`query`中的元素会匹配`url`后的查询参数`?key1=value1&key2=value2`，并且任意顺序都可以被匹配到，以下两组URL都会被匹配到：
- `/with/parameters?search=search+terms&filter=month`
- `/with/parameters?filter=month&search=search+terms`

如果查询参数只有KEY没有VALUE，则匹配时URL需要给定KEY，Payload如下：
``` yml
- request:
    url: ^/with/parameters$
    method: GET
    query:
        search:
        filter: month
```

则以下两组URL都会被匹配到：
- `/with/parameters?search&filter=month`
- `/with/parameters?search=&filter=month`

**示例四：POST时指定发送的Payload**
``` yml
- request:
    url: ^/path/to/something$
    method: POST
    headers:
        Content-Type: application/json
        authorization-basic: "bob:password" 
        x-custom-header: "^this/is/\d/test"
    post: >
        {"key": value, "data": "content"}
  response:
    headers:
        Content-Type: application/json
    status: 201
    body: Your request was successfully processed!

```

**示例五：返回的Response是Json文件**
``` yml
- request:
    method: GET
    url: /users/456
  response:
    status: 200
    headers:
        Content-Type: application/json
    file: json/users-response.json
```

其中的file路径是当前YAML文件的相对路径。


Request和Response的Payload除了使用JSON外，还可以使用XML格式或直接使用文本，另外配置文件除了使用YAML格式外，也可以合适JSON格式。更多示例可以参考[github stubby4j](https://github.com/azagniotov/stubby4j)。

----
References
* [github stubby4j](https://github.com/azagniotov/stubby4j)