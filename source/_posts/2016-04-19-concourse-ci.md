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

[Jenkins](https://jenkins.io/)作为使用最广泛，用户量最大的CI工具，必定有其可取之处，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得很出色，而且简单易学，入门上手快，当然Jenkins的优势还有很多，之前的项目上都一直在使用Jenkins，对于大多项目来说是完全满足条件的。

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

[Travis CI](https://travis-ci.com/)其实各方面也都挺不错的，没有Snowflake配置，使用.travis.yml文件配置，在容器中运行Builds，与Github集成度很好，支持PR。

但是也有一些缺点：如它不支持pipeline，只能支持简单的构建; 并且如果CI跑不过，需要设置多个Debug提交点来找到问题所在; 另外Travis CI是由个人发起的项目，并且目前只对开源软件免费。

总体说来，对于公司的项目，更倾向于选择GoCD或Jenkins这样的产品，个人的开源项目倒是可以通过Travis CI快速搭建来感受一下。

----

### Concourse Concepts
Concourse的核心概念: resources, jobs, tasks. 通过这三个核心模块可以对任何的Pipeline进行建模，从简单的unit->integration->deploy->ship到复杂的多基础设施, fanning out/in等。

1. Resources: 就是资源，最重要的概念之一，Concourse把需要交互的对象都视作一种资源，比如Github上的某个工程代码，AWS的S3存储，以及其他的外部服务等。更多资源类型可参见[Customer Resource Types](http://concourse.ci/resource-types.html)
![](/assets/concourse-ci/concourse_resources.png)

2. Jobs: Job是某个有计划有条件的工作任务，描述了一些依赖资源或手动触发的行为，当提交了代码就触发Build，或Unit Test通过后触发了Intg Test等。

3. Tasks: Task就是在隔离环境中依赖于资源的某个脚本执行，如build, test等，Job其实封装了Task，Job更侧重于描述什么情况下发生，而Task重点描述发生什么。

----

### Concourse Architecture
Concourse架构属于一种简单的分布式系统，其三大核心部件分别为: `ATC`, `TSA`和`Workers`，接下来将分别进行介绍。

![](/assets/concourse-ci/architecture.png)

#### ATC: web UI & build scheduler
ATC主要用于运行Web UI和API以及所有Pipeline构建计划的，属于Concourse的心脏，占据了极其重要的位置。采用PostgreSQL数据库存储Pipeline数据和构建日志。

多个ATCs可以作为一个集群运行，各个ATC都共享一个数据库，ATC通过加锁机制在集群之间同步与传输数据。

ATC默认监听`8080`端口，通常与`TSA`一起处于负载均衡(Load Balancer)之后，为了实现正常拦截([Intercept](http://concourse.ci/fly-intercept.html))构建(Build)的功能，需要确保Load Balancer被正确配置到了TCP或SSL转发，而非HTTP或HTTPS。

#### TSA: worker registration & forwarding
TSA是`ATC`定制的SSH服务器，仅用于安全地注册`Workers`，仅支持两个命令`register-worker`和`forward-worker`。

- register-worker命令用于为ATC直接注册在同一私有网络中运行的worker。
- forward-worker命令用于通过TSA反向隧道worker的地址，然后为ATC注册转发连接。这样只要workers能够连接到TSA，就可以运行在任意网络且安全地实现注册功能，ATC也就可以安全地连接到worker，该方式把worker与外界环境进行隔离，只有通过授权后才能访问，从而提高了其安全性。

TSA默认监听`2222`端口，通常与ATC协同工作运行在Load Balancer的之后。

#### Workers: container runtime & cache management
Workers可以认为是一台通过`TSA`进行自注册的正在运行[Garden](https://github.com/cloudfoundry-incubator/garden)和[Baggageclaim](https://github.com/concourse/baggageclaim)服务的机器。

Workers在自己所属机器上并没有配置重要的状态什么的，所有的内容都运行在容器中，更不需要关心相关依赖包安装到主机上在，这就是workers区别于其他非容器化(non-containerized)的CI解决方案，特别是当workers中的依赖包的状态成为了pipeline正常工作与否的关键因素，容器化显得尤为重要。

每一个worker通过TSA注册自己，从而可以被Concourse集群发现并使用。Workers中的`Garden`默认监听`7777`端口，`Baggageclaim`默认监听`7788`端口。如果都在`ATC`可达的同一个私有网络中，那么workers会绑定到所有地址`0.0.0.0`，并且会直接注册自己，否则会绑定到`127.0.0.1`，通过`TSA`进行转发。

以上就是Concourse的架构，更多细节可以参见[Concourse Architecture](http://concourse.ci/architecture.html)

----

### Concourse Impacts
之所以要介绍Concourse，是因为Concourse带了一些新思路和想法，提供了另一个看待CI/CD的视角。

- Pluggable Resource Interface: 把操作对象看待资源，提供对资源访问的可接口，达到松耦合的目的
- Running builds in Containers Natively: 本身就支持在容器中运行构建，隔离不同环境
- Zero Snowflake-able Configuration: 没有雪花式的配置，可以重用相同的配置，通过版本控制管理，快速恢复和移植
- Submitting builds from the local file system up to run in CI: 通过本地配置文件提交构建到CI中运行，每个项目单独管理Build，降低维护成本
- [Cloud Foundry](https://www.cloudfoundry.org/): Run on diff architectures, diff platforms, against varying underlying IaaSs: Cloud Foundry是一款PaaS平台即服务产品，为了解决Cloud Foundry的CI/CD问题才开发出Concourse，些类项目需要运行在不同的架构，不同的平台以及不同的基础设施中，因此Pipeline会相当复杂，而Concourse在配置这样复杂的Pipeline的时候表现得更加令人满意。

总得来说，Concourse带来了新的思路，并且与当下流行的Container技术结合，同样在支持PaaS项目的CI/CD时显得略胜一筹。

----

### Using Concourse

使用Concourse不会过多作介绍，后续会有专门针对如何使用Concourse的详细教程，本博客中只简单介绍一下流程:

#### Step1. Install and Setup
通常安装和运行Concourse有三种方式，任选一种方式尝试安装并启动：

- Local VM with Vagrant
- Standalone Binaries
- Clusters with BOSH

首先我们在本地采用最快捷的Vagrant方式安装，运行以下命令：

```
vagrant init concourse/lite  # creates ./Vagrantfile
vagrant up  # downloads the box and spins up the VM
```

然后Concourse服务已经开始运行了，通过[192.168.100.4:8080](http://192.168.100.4:8080)地址进行访问。
![](/assets/concourse-ci/no_pipeline.png)

如果提示需要更新升级，可以尝试运行以下命令：

```
vagrant box update --box concourse/lite # gets the newest Vagrant box
vagrant destroy                         # remove the old Vagrant box
vagrant up                              # re-create the machine with the newer box
```

但针对线上环境，不推荐Vagrant方式安装，Vagrant用于学习目的，快速掌握Concourse工作方式还是不错的选择，如果针对线上产品的项目，可以尝试采用后两种方式安装，更多安装介绍请参见[Concourse Installing](http://concourse.ci/installing.html)。

当安装Concourse完成后，还需要在本地下载[the Fly CLI](http://concourse.ci/fly-cli.html). 也可以访问Concourse主界面，然后点击Fly CLI链接进入下载页。

针对Linux和Mac OS X系统，首先需要给下载的FLY CLI文件添加执行权限，然后安装到系统并添加到$PATH中：

```
chmod +x fly
install fly /usr/local/bin/fly
```

#### Step2. Using *.yml to describe pipeline
为了创建一个Pipeline，首先在创建名为`hello.yml`的文件，并写入以下内容:

```
jobs:
- name: hello-world
  plan:
  - task: say-hello
    config:
      platform: linux
      image_resource:
        type: docker-image
        source: {repository: ubuntu}
      run:
        path: echo
        args: ["Hello, world!"]
```

关于定义Pipeline的更多内容可参见[Pipelines](http://concourse.ci/pipelines.html)。

#### Step3. Push to Concourse

如果使用的是Vagrant安装方式，我们可以尝试登录到本地VirutalBox中：

```
fly -t lite login -c http://192.168.100.4:8080
```
当前已经保存了名为`lite`的目标，会在以后的多个命令行中使用，`-t`代表目标名(Target Name)。

当准备好`hello.yml`后，可以通过以下命令设置Pipelilne：

```
fly -t lite set-pipeline -p hello-world -c hello.yml
```

然后刷新Concourse主页面，可以看到已经设置好一个简单的Hello World的Pipeline了。
![](/assets/concourse-ci/hello_demo_pipeline.png)

该默认配置是暂停Pipeline，可以通过界面启动，也可以通过命令行方式启动：

```
fly -t lite unpause-pipeline -p hello-world
```

也可以通过命令查看当前Pipeline的配置：

```
fly -t lite get-pipeline -p hello-world
```

该Pipeline非常简单，只有单一的`Job`，整个计划中只有一个`Task`，可以看到`Task`执行过程的快照：

![](/assets/concourse-ci/hello_run_build.png)

更多示例教程可以参见[Concourse官网Demo](http://concourse.ci/tutorials.html)或[Github的Concourse教程](https://github.com/starkandwayne/concourse-tutorial)。

### Concourse Assess
虽然列举出了很多Concourse的优点和创新思路，但也有一些Concerns:

- Concerns 1: 与其他CI/CD以GUI进行配置的方式不同，虽然Build和Pipeline的UI可视化效果很不错，但在配置上采用了YML文件，且需要使用FLY命令行的方式进行交互，因此要求记住一些规则、关键字和相关命令，这样势必增加了学习成本。
- Concerns 2: 因为Concourse作为新产品新工具，稳定性、易用行、扩展性等各方面还有待市场和用户的大量验证，当然，如果Concourse确实能够带来更多方便和更多新思路，并且解决了用户的真正痛点问题，那么，相信其未来一定是非常光明的。

至于目前选择用哪款工具，其实需要根据团队和项目的情况来平衡选择的，有兴趣喜欢尝鲜的同学倒是可以折腾下，还是很有意思的。

----

[1] https://concourse.ci
[2] https://www.go.cd/
[3] https://jenkins.io/
[4] https://travis-ci.com/
[5] https://en.wikipedia.org/wiki/First-class_citizen
[6] https://github.com/starkandwayne/concourse-tutorial