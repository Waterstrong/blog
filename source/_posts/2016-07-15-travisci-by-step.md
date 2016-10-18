---
title: Travis CI Step by Step
date: 2016-07-15 21:57:02
category: Tools
tags: [DevOps, CI/CD, TravisCI, GitHub]
thumbnailImage: /assets/travisci-by-step/travis_ci_home.png
description: Travis CI是一款提供托管与分布式持续集成服务的CI工具，与GitHub高度集成，能够构建和测试托管在GitHub上的软件项目，主要为开源免费提供轻量级可定制化的持续持续集成环境和服务。
published: true
---

## Travis CI介绍
> Travis CI is a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub.

Travis CI是一款提供托管与分布式持续集成(Continuous Integration)服务的CI工具，与GitHub高度集成，能够构建和测试托管在GitHub上的软件项目。

![](/assets/travisci-by-step/travis_ci_home.png)
Travis CI主要为开源免费提供轻量级可定制化的持续持续集成环境和服务，而对于非开源项目，会按照相应的标准收取一定的费用。Travis CI不仅支持多种语言，而且支持在容器中运行Builds，与GitHub集成度很好，支持Pull Request等。一般通过简单配置`.travis.yml`文件即可使用，也省去了自己搭建和维护CI服务器的繁琐工作，但它不支持pipeline，只能支持简单的构建。

## Travis CI集成
Travis CI与GitHub集成只需要简单的几步即可，通过访问[GitHub Integrations](https://github.com/integrations)搜索`Travis CI`选择进入子页面，可以看到对Travis CI的集成的基本介绍。
![](/assets/travisci-by-step/integrations.png)

#### GitHub登录授权
Travis CI与GitHub集成需要登录GitHub帐号，一般有两种入口方式：
1. 在[Travis CI Integration](https://github.com/integrations/travis-ci)页面中点击`Add to GitHub`->`Authorize application`进行授权。
2. 对于**Public**项目直接访问[travis-ci.org](https://travis-ci.org/)并登录GitHub帐号授权，而**Private**项目应访问[travis-ci.com](https://travis-ci.org/)。
![](/assets/travisci-by-step/authorize_travis_ci.png)

另外，若需要管理Travis CI的权限(Revoke/Grant Access)，可以在登录[GitHub](https://github.com/)后点击头像下拉菜单中的[Settings](https://github.com/settings/)，选择左边导航栏中的[OAuth applications](https://github.com/settings/applications)，进入`Travis CI`应用进行操作，对于未授权的Organization，通常需要手动点击`Grant access`确认才能授权Travis CI访问。
![](/assets/travisci-by-step/oauth_travis_ci.png)

#### 激活GitHub仓库
Travis CI触发Build的原理是基于GitHub的Service Hook钩子实现，而需要Travis CI应用到自己的项目，需要在Travis CI的[Profile](https://travis-ci.org/profile/) (`Yourname`->`Accounts`)中选择对应的Repository并开启Hook，若没有看到项目列出，请点击`Sync account`同步一下。
![](/assets/travisci-by-step/hook_switch.png)

若需要设定某些配置项或设置环境变量，可以点击齿轮状的按钮进行设置页面，更多可参阅[Travis CI Docs](https://docs.travis-ci.com/user/cron-jobs/)。
![](/assets/travisci-by-step/travis_ci_settings.png)

#### 配置TRAVIS YML文件
Travis CI支持多种语言的项目，只需要在项目目录下新建文件`.travis.yml`，加入几行配置即可，比如选择[Java](https://docs.travis-ci.com/user/languages/java)项目(Gradle+Java8)，一个简单的示例配置如下：
``` yml
language: java
install: true
script: ./gradlew clean build
before_cache:
  - rm -f $HOME/.gradle/caches/modules-2/modules-2.lock
cache:
  directories:
    - $HOME/.gradle/caches/
    - $HOME/.gradle/wrapper/
jdk:
  - oraclejdk8
```

也可以针对多个Java版本进行测试，保证项目的兼容性，只需要设置jdk即可：
``` yml
jdk:
  - oraclejdk8
  - oraclejdk7
  - openjdk6
```

再比如[Javascript Node.js](https://docs.travis-ci.com/user/languages/javascript-with-nodejs)项目(NodeJs+Gulp+Bower+Redis)的简单示例配置如下：
``` yml
language: node_js
node_js:
  - "6"
before_script:
  - npm install -g gulp bower
  - npm install
  - bower install
script: gulp
services:
  - redis-server
```

另外，也支持[Installing Dependencies](https://docs.travis-ci.com/user/installing-dependencies/), [Setting up Databases](https://docs.travis-ci.com/user/database-setup/), [Deployments and Uploads](https://docs.travis-ci.com/user/deployment/)以及[Configuring Build Notifications](https://docs.travis-ci.com/user/notifications/)等。更多配置项详情请参阅[Getting Started](https://docs.travis-ci.com/user/getting-started/)。

#### 触发Build构建
当所有配置完成后，下一次提交代码时就会触发Build，可以看到Build的一些详细信息，包括Build号，Commit号，Elapsed用时，Log日志等信息，如果成功就显示绿色，失败则显示红色。
![](/assets/travisci-by-step/trigger_build.png)

除了查看Current当前的Build，也可以选择查看Branches分支状态或Build History历史记录等。

#### 关联Build状态
为了显示指明项目与Travis CI的关系与状态，可以在项目README.md中关联Build状态图标，点击项目右边的状态图片![](/assets/travisci-by-step/passing_status.svg)，然后会弹出一个Status Image链接的页面，选择Markdown格式，然后复制链接到项目的README.md中即可。
![](/assets/travisci-by-step/link_build_status.png)

Build状态标识的Markdown格式为：
``` md
[![Build Status](https://travis-ci.org/GITHUB_NAME/PROJECT_NAME.svg?branch=master)](https://travis-ci.org/GITHUB_NAME/PROJECT_NAME)

# E.g. GITHUB_MAME=tw-wee, PROJECT_NAME=qas-service
[![Build Status](https://travis-ci.org/tw-wee/qas-service.svg?branch=master)](https://travis-ci.org/tw-wee/qas-service)
```

## The End
总体说来，对于公司或企业项目，更倾向于选择GoCD或Jenkins这样的产品，个人或社区开源项目建议采用Travis CI来快速实现持续集成服务。

----
References

* [Travis CI Docs](https://docs.travis-ci.com/)
* [Travis CI Wiki](https://en.wikipedia.org/wiki/Travis_CI)