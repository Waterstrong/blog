---
title: Concourse CI 介绍
date: 2016-04-19 13:02:02
category: Tools
tags: [DevOps, CI/CD, Concourse, Jenkins, TravisCI, GoCD]
description: 目前主流的CI/CD工具包括Concourse CI, Jenkins, Travis CI和GoCD，它们各自到底有什么优缺点，Concourse CI有什么优势和亮点能够跻身Tech Radar?
---

目前主流的CI/CD工具包括Concourse CI, Jenkins, Travis CI和GoCD，它们各自到底有什么优缺点，Concourse CI有什么优势和亮点能够跻身April '16的[ThoughtWorks Tech Radar](https://www.thoughtworks.com/radar/tools/concourse-ci)？

### Advantages of CI/CD
首先还是快速介绍一下CI/CD，特别是为什么要采用CI/CD，有什么样的优势，只有在有意义的前提下，使用工具才能发挥作用，并且解决项目开发中的痛点问题。

#### Continuous Integration
Continuous Integration(持续集成). Integrating, building, and testing code within the development environment.

#### Continuous Delivery
Continuous Delivery(持续交付). A software development discipline, build software that can be released to production at any time.

#### CI/CD的好处
1. Reduced Deployment Risk，降低部署风险。快速提交小部分修改进行部署和集成，从而降低了出现错误的概率，即使出现错误也能快速定位问题并修复。
2. Believable Progress，可信的进度。如果直接部署到线上环境中，项目进展以及完成度相对于开发人员自己声称已经完成要更加的有可信度。
3. User Feedback，用户反馈。众所周知，项目开发中最大的风险就是开发的软件不被用户接受，这样的软件是没有太大的用处和意义的，尽早和更加频繁地交付给用户并且快速获得用户反馈来获取有价值的内容，从而保证了开发的软件是被用户接受的。

有关CI/CD的更多详细解释可以参见Martin Fowler博客文章[ContinuousDelivery](http://martinfowler.com/bliki/ContinuousDelivery.html)。

----

### What is Concourse?

> Concourse is a `CI/CD tool` that treats `build pipelines and artifacts as first-class citizens`.
> It enables builds that `run in containers`, has a `clean, usable UI and discourages snowflake` build servers.
> It aims to provide an `expressive system` with as `few distinct moving parts` as possible.

先感受一下Concourse的界面，这是Concourse项目本身的Pipelines:
![](/assets/concourse-ci/concourse_pipeline.png)

**Concourse CI是一款CI/CD工具，把构建pipeline和artifacts当作first-class citizens(可译作: 第一类公民)。**

> **First-class Citizens:** In programming language design, a first-class citizen (also type, object, entity, or value) in a given programming language is an entity which supports all the operations generally available to other entities. These operations typically include being passed as an argument, returned from a function, and assigned to a variable.

*第一类公民：* 即支持其他实体所有操作的实体，比如能够在运行时被动态创建，能够作为参数或返回值直接被其他实体消费或生成。举个例子，在C语言中，function就不是第一类公民，而在Javascript中function是第一类公民。其中，实体是指各种各样的数据类型和值，比如对象、类、函数、字面量等。

**Concourse CI本身就与容器结合，Build构建在容器中运行，隔离各个环境，避免不同环境之间相互污染情况发生。**

**表现系统意味着有更简洁清晰可用的UI，而尽量少的移动部件意味着模块组件统一化，并且不会有雪花式的配置，Concourse CI采用YAML文件配置Pipeline，并且通过版本控制管理起来，很容易地实现移植和恢复。**

下图中展示了一个标准的Pipeline示例，其中的黑色框元素代表资源(Resources)，彩色框元素代表Jobs，会有不同的颜色代表Build的状态，流线代表了依赖和执行顺序，如Integration需要前面所有的Jobs执行成功并且提供相关Resources才能正确触发并执行。
![](/assets/concourse-ci/standard_pipeline_demo.png)

对Jobs简单的YML配置示例:
![](/assets/concourse-ci/config_yml_demo.png)


----

### Why is Concourse?
为什么会出现Concourse呢？它能带来什么新思路呢？相比目前已有的CI/CD工具有什么区别呢？

- Requires a CI/CD Tool, 首先当然是提供CI/CD的基本功能，需要一款CI/CD工具来解决项目开发中的一系列问题。
- Concourse vs GoCD/Jenkins/Travis CI，除此之外，还需要弥补目前CI/CD工具的一些不足，并加入一些新元素。

#### vs. GoCD
![](/assets/concourse-ci/gocd_pipelines.png)

GoCD属于老大级的，需要安装Server和Agent，其设计之初就是为了满足CI/CD的需求，把Build Pipelines和Artifacts作为First-class Citizens，因此也更好地支持自动化和流线式的build-test-release这样的CI/CD周期运作，同时，也支持更加复杂的工作流(Workflows)，包括并行和串行的，即使是很复杂的工作流也同样可以非常清晰直观地展示出来。

除此之外，当然GoCD还有很多其他的优点，并且目前项目组也在使用。但是美中不足是GoCD配置操作的GUI很不人性化，使用过的人都知道，查看Jobs运行情况与配置Jobs的切换很麻烦，需要到首页分别点击Pipelines和Settings，而且每一项菜单层级很深，操作不友好，GoCD在架构设计上分为Pipelines -> Stages -> jobs -> tasks，层级嵌套显得有些复杂，不过这样的划分还好，只要操作上更加人性化就可以了。

更多关于GoCD的说明请参见[More about GoCD](https://www.go.cd/)

#### vs. Jenkins


#### vs. Travis CI



### Concourse Concepts
End Goal: To provide an expressive system with as few distinct moving parts as possible

### Concourse Architecture

### Concourse Impacts
Bringing some interesting new ideas
- Pluggable Resource Interface
- Running builds in Containers Natively
- Zero Snowflake-able Configuration
- Submitting builds from the local file system up to run in CI

### Using Concourse

----

[1] https://concourse.ci
[2] https://en.wikipedia.org/wiki/First-class_citizen
[3] https://www.go.cd/
[4] https://jenkins.io/