---
title: Travis CI by Step
date: 2016-07-15 21:57:02
category: Tools
tags: [DevOps, CI/CD, TravisCI, Github]
description: Travis CI是一款提供托管与分布式持续集成服务的CI工具，与Github高度集成，能够构建和测试在Github上的软件项目。主要为开源免费提供轻量级可定制化的持续持续集成环境和服务。Travis CI不仅支持多种语言，而且支持在容器中运行Builds，通过简单配置.travis.yml文件即可使用，也省去了自己搭建和维护CI服务器的繁琐工作。
published: true
---

## Travis CI介绍
Travis CI是一款提供托管与分布式持续集成服务的CI工具，与Github高度集成，能够构建和测试在Github上的软件项目。

> Travis CI is a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub.

Travis CI主要为开源免费提供轻量级可定制化的持续持续集成环境和服务，而对于非开源项目，会按照相应的标准收取一定的费用。Travis CI不仅支持多种语言，而且支持在容器中运行Builds，与Github集成度很好，支持Pull Request等。一般通过简单配置`.travis.yml`文件即可使用，也省去了自己搭建和维护CI服务器的繁琐工作，但它不支持pipeline，只能支持简单的构建。	

## Travis CI集成

#### Github登录授权
OAuth applications
![](/assets/travisci-by-step/authorisation.png)

#### 激活Github仓库
Travis CI触发Build的原理是基于Github的Service Hook实现，而需要Travis CI应用到自己的项目，需要在Travis CI的[Profile](https://travis-ci.org/profile/) (`Yourname`->`Accounts`)中选择对应的Repository并开启Hook。
![](/assets/travisci-by-step/hook_switch.png)

#### 配置`.travis.yml`文件


#### 触发Build构建
![](/assets/travisci-by-step/trigger_build.png)


#### 关联Build状态


## 结束语
总体说来，对于公司或企业项目，更倾向于选择GoCD或Jenkins这样的产品，个人或社区开源项目建议采用Travis CI来快速实现持续集成服务。

----
References

* [Travis CI Docs](https://docs.travis-ci.com/)
* [Travis CI Wiki](https://en.wikipedia.org/wiki/Travis_CI)