---
title: JBoss Tips in Practice
date: 2016-04-20 22:23:50
category: Frameworks
tags: [Java, Web容器, JBoss, EJB, Servlet, JSP]
description: JBoss是众多Java EE容器中的一个，它不但是Servlet容器，而且是EJB容器，弥补了Tomcat只是一个Servlet容器的缺憾。为了实现一些特定的功能, 需要针对JBoss的一些配置, 将列出一些Tips供使用参考。
---

JBoss是众多Java EE容器中的一个，它不但是Servlet容器，而且是EJB容器，弥补了Tomcat只是一个Servlet容器的缺憾。为了实现一些特定的功能, 需要针对JBoss的一些配置, 以下列出一些Tips供使用参考。
<!-- more -->

----

## Tip1: IP/域名黑白名单
针对JBoss EAP5，为了实现Web容器级别的IP/域名黑白名单，需要到JBoss的server.xml文件中配置相应Valve规则并重启JBoss后生效。

在文件`jboss-as/server/<instance_name>/deploy/jbossweb.sar/server.xml`中的`<Host></Host>`内添加如下配置:
```
<Engine>
    <Host>
        .....

        <Valve className='org.apache.catalina.valves.RemoteAddrValve'
        allow='192.168.[0-1].*, *.testing.com'
        deny='127.0.0.1'/>
    </Host>
</Engine>
```
其中`allow`表明允许的IP地址正则表达式，`deny`表明拒绝的IP地址正则表达式，特别注意的是逗号(`,`)会被解析成`或`，因此IP Regex中一定不要包含`逗号`。

当然，除了可以设置服务器级别`server-level`，也可以设置应用层级别`application-level`，更多关于Web Server的配置可参见[Configuring the Web Server](http://www.datadisk.co.uk/html_docs/java_app/jboss5/jboss5_web_server.htm)。

另外补充一下Valve的相关解释:
> Valves are similar to filters, they can intercept any incoming and outgoing request. Valves are managed by the Engine, they access incoming/outgoing requests before they are handled by the servlet and JSP processing logic. Logically they can also be applied on a virtual host or web application basis.
Valves can add the following functionality:
- Access logging
- Single sign-on for all Web applications
- Request filtering/blocking by IP address and or hostname
- Dumping of incoming/outgoing request headers for debugging purposes

> Valves are nested components in the component model, they use the <valve> XML element in the server.xml file, they can be placed in the <Engine>, <Host> or <Context> containers. The Java programming interface org.apache.catalina.Valve is used and well documented.

想了解更多其他内容及详细解释请参见[Advanced Tomcat Features](http://www.datadisk.co.uk/html_docs/java_app/tomcat6/tomcat6_advanced.htm)。

----

## Tip2: 自动清除work目录
针对JBoss EAP5:
> The work directory:
- Directory where compiled JSP .java and .class files reside
- Also contains cached TLDs
- Very useful for debugging problems in JSPs
Java ServerPages (.jsp files) are automatically compiled into Java Servlets (.java file) and then into Java byte-code (.class files) by Tomcat (the embedded servlet engine running within JBoss AS).

JBoss中的work目录是工作目录，即把jsp转换为class文件的工作目录, 其工作原理是当浏览器访问某个jsp页面时，JBoss会在work目录里把这个jsp页面转换成.java文件，然后编译为.class文件，最后容器通过ClassLoader类把这个.class类装载入内存，进行响应客户端的工作。

通常情况下会定时检查容器内的jsp文件，读取每个文件的属性，当发现某个jsp文件发生改变时(文件的最后修改时间与上次检查时不相同)，容器会重新转换、编译这个jsp文件, 但是检查是定时的不是实时的，因此jsp文件修改后需要几分钟的时间来等修改过的jsp生效。为了即刻生效，通常的做法是重启JBoss之前或在修改jsp页面后立即清除work目录里的文件。

一般情况下, 当停止JBoss服务时对work目录进行一次清理, 最简单快速的做法就是在JBoss相应的目录下配置自动清理选项:
修改文件`jboss-as/server/<instance_name>/deployers/jbossweb.deployer/META-INF/war-deployers-jboss-beans.xml`中的:
```
<!--Flag to delete the Work Dir on Context Destroy -->
<property name="deleteWorkDirOnContextDestroy">false</property>
```
为
```
<!--Flag to delete the Work Dir on Context Destroy -->
<property name="deleteWorkDirOnContextDestroy">true</property>
```

----





