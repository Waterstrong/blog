---
title: Concourse CI 介绍
date: 2016-04-19 13:02:02
category: Tools
tags: [DevOps, CI/CD, Concourse, Jenkins, TravisCI, GoCD]
description: 目前主流的CI/CD工具包括Concourse CI, Jenkins, Travis CI和GoCD，它们各自到底有什么优缺点，Concourse CI有什么优势和亮点能够跻身Tech Radar?
---

目前主流的CI/CD工具包括Concourse CI, Jenkins, Travis CI和GoCD，它们各自到底有什么优缺点，Concourse CI有什么优势和亮点能够跻身April '16的[ThoughtWorks Tech Radar](https://www.thoughtworks.com/radar/tools/concourse-ci)？

### 快速回顾CI/CD
首先还是快速介绍一下CI/CD，特别是为什么要采用CI/CD，有什么样的优势，只有在有意义的前提下工具才能发挥作用，并且解决项目开发中的痛点问题。

#### Continuous Integration
Continuous Integration(持续集成). Integrating, building, and testing code within the development environment.

#### Continuous Delivery
Continuous Delivery(持续交付). A software development discipline, build software that can be released to production at any time.

#### CI/CD的好处
1. Reduced Deployment Risk，降低部署风险，快速提交小部分修改进行部署和集成，从而降低了出现错误的概率，即使出现错误也能快速定位问题并修复。
2. Believable Progress，可信的进度，如果直接部署到线上环境中，项目进展以及完成度相对于开发人员自己声称已经完成要更加地有可信度。
3. User Feedback，用户反馈，众所周知，项目开发中最大的风险就是开发的软件不被用户接受，这样的软件是没有太大的用处和意义的，迟早和更加频繁地交付给用户并且快速获得用户反馈来获取有价值的内容。

有关CI/CD的更多详细解释可以参见Martin Fowler博客文章[ContinuousDelivery](http://martinfowler.com/bliki/ContinuousDelivery.html)。

### What is Concourse?

> Concourse is a `CI/CD tool` that treats `build pipelines and artifacts as first-class citizens`.
> It enables builds that `run in containers`, has a `clean, usable UI and discourages snowflake` build servers.
> It aims to provide an `expressive system` with as `few distinct moving parts` as possible.

ConcourseCI是一款CI/CD工具，把构建pipeline和artifacts当作first-class citizens，可以译为第一类公民。

> **First-class Citizens:** In programming language design, a first-class citizen (also type, object, entity, or value) in a given programming language is an entity which supports all the operations generally available to other entities. These operations typically include being passed as an argument, returned from a function, and assigned to a variable.

ConcourseCI本身就与容器结合，构建在容器中运行，隔离各个环境，避免不同环境之间相互污染情况发生。

表现系统意味着有更简洁清晰可用的UI，而尽量少的移动部件意味着模块组件统一化，并且不会有雪花式的配置，ConcourseCI采用YAML文件配置Pipeline，并且通过版本控制管理起来，移植和恢复都很容易。


### Why Concourse?
- Requires a CI/CD Tool
- Concourse vs Jenkins/Travis CI/GoCD

#### vs. Jenkins


#### vs. Travis CI


#### vs. GoCD


### Concepts


### Impact
Bringing some interesting new ideas
- Pluggable Resource Interface
- Running builds in Containers Natively
- Zero Snowflake-able Configuration
- Submitting builds from the local file system up to run in CI

[1] https://concourse.ci
[2]
[3] https://en.wikipedia.org/wiki/First-class_citizen