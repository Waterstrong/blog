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

有关CI/CD的更多详细解释可以参见Martin Fowler博客文章[Continuous Delivery](http://martinfowler.com/bliki/ContinuousDelivery.html)。

----

### What is Concourse?

> [Concourse](https://concourse.ci) is a `CI/CD tool` that treats `build pipelines and artifacts as first-class citizens`.
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

[GoCD](https://www.go.cd/)属于老大级的，需要安装Server和Agent，其设计之初就是为了满足CI/CD的需求，把Build Pipelines和Artifacts作为First-class Citizens，因此也更好地支持自动化和流线式的build-test-release这样的CI/CD周期运作，同时，也支持更加复杂的工作流(Workflows)，包括并行和串行的，即使是很复杂的工作流也同样可以非常清晰直观地展示出来。

除此之外，当然GoCD还有很多其他的优点，并且目前项目组也在使用。但是美中不足是GoCD配置操作的GUI很不人性化，使用过的人都知道，查看Jobs运行情况与配置Jobs的切换很麻烦，需要到首页分别点击Pipelines和Settings，而且每一项菜单层级很深，操作不友好，GoCD在架构设计上分为Pipelines -> Stages -> jobs -> tasks，层级嵌套显得有些复杂，不过这样的划分还好，只要操作上更加人性化就可以了。

而Concourse针对GUI这一点进行了一些改进，并且引入了一种YML文件配置机制来实现对Job的配置，同样支持复杂的Workflow，也将Build Pipelines和Artifacts作为First-class Citizens，并且设计之初本身就与容器技术结合，每个Build都在Container中运行。

#### vs. Jenkins
![](/assets/concourse-ci/jenkins_dashboard.png)
![](/assets/concourse-ci/jenkins_plugins.png)

[Jenkins](https://jenkins.io/)作为使用最广泛，用户量最大的CI工具，必定有其可取之处，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得很出色，而且简单易学，入门上手快，当然Jenkins的优势还有很多，之前的项目上都一直在使用Jenkins，对于大多项目来说是完成满足条件的。

*但Jenkins也有其缺点，比如：*
- 在配置Shell命令时，如果Pipeline规模扩大，构建和部署环境增多，那么就会复制粘贴很多这样的Shell命令，称为雪花式(Snowflakes)配置，增加维护成本；
- 另外，Jenkins定义Job的顺序是以Job为关注点，从全局出发，比如定义A Job的前置Job是B，后置Job是C，当Jobs顺序情况变得复杂就很难再梳理清楚了；
- Jenkins并未将Build Pipelines和Artifacts视作First-class Citizens，如果需要实现Continuous Delivery是需要借助插件完成，而Jenkins本身并不直接支持CD的；
- 此外，虽然Jenkins的插件生态系统管理得很好，一旦Workspace中有很多的插件，难免会造成一些插件问题导致Build环境被污染。

**针对Jenkins的一些问题，Concourse进行了一些改进，比如：**
- Snowflakes的情况就采用统一配置YML文件来解决，各Task各工程自己维护，有重复的内容通过提取文件多处引用即可；
- 通过VCS将这样配置文件进行版本控制管理，在恢复或移植时更加方便，虽然Jenkins和GoCD也有XML配置文件，但通常用于备份，一般的做法也不会进行版本控制，更不会直接去修改文件来实现Pipeline配置；
- Concourse对于每个Job只定义有效的输入，即哪个Job在什么情况下输出的什么的资源是可以触发当前Job的，即使复杂的Pipeline顺序出现时，配置也很方便，每个Job只关心自己的有效输入，局部优化达到了全局优化，不会造成混乱的感觉；
- 特别指出的是Concourse的每一个Job构建都在独立的Container中Build，对其他的环境没有影响。

#### vs. Travis CI
[Travis CI](https://travis-ci.com/)

----

### Concourse Concepts
End Goal: To provide an expressive system with as few distinct moving parts as possible

----

### Concourse Architecture

----

### Concourse Impacts
Bringing some interesting new ideas
- Pluggable Resource Interface
- Running builds in Containers Natively
- Zero Snowflake-able Configuration
- Submitting builds from the local file system up to run in CI
- [Cloud Foundry](https://www.cloudfoundry.org/): Run on diff architectures, diff platforms, against varying underlying IaaSs

----

### Using Concourse

----

[1] https://concourse.ci
[2] https://www.go.cd/
[3] https://jenkins.io/
[4] https://travis-ci.com/
[5] https://en.wikipedia.org/wiki/First-class_citizen