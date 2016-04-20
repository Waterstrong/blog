---
title: JBoss的一些功能Tips
date: 2016-04-20 22:23:50
category: Web Container
tags: [Web容器, JBoss, EJB, Servlet, JSP]
description: JBoss是众多Java EE容器中的一个，它不但是Servlet容器，而且是EJB容器，弥补了Tomcat只是一个Servlet容器的缺憾。为了实现一些特定的功能, 需要针对JBoss的一些配置, 将列出一些Tips供使用参考。
---

JBoss是众多Java EE容器中的一个，它不但是Servlet容器，而且是EJB容器，弥补了Tomcat只是一个Servlet容器的缺憾。为了实现一些特定的功能, 需要针对JBoss的一些配置, 以下列出一些Tips供使用参考:

## Tip1: IP/域名黑白名单


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
修改文件`server\<instance_name>\deployers\jbossweb.deployer\META-INF\war-deployers-jboss-beans.xml`中的:
```
<!--Flag to delete the Work Dir on Context Destroy -->
<property name="deleteWorkDirOnContextDestroy">false</property>
```
为
```
<!--Flag to delete the Work Dir on Context Destroy -->
<property name="deleteWorkDirOnContextDestroy">true</property>
```




