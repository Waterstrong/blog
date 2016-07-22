---
title: Jenkins Step by Step
date: 2016-07-06 21:30:44
category: Tools
tags: [DevOps, CI/CD, Jenkins]
description: Jenkins是一款开源的跨平台的可扩展的持续集成工具。作为目前使用最广泛，用户量最大的CI工具，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得较为出色，而且简单易学，入门上手快。
published: true
---

## Jenkins Introduction
Jenkins是一款开源的跨平台的可扩展的持续集成(Continuous Integration)工具。作为目前使用最广泛，用户量最大的CI工具，无论是在GUI操作上，插件生态系统管理，稳定性、可靠性、功能性以及扩展性等方面都表现得较为出色，而且简单易学，入门上手快，当然Jenkins的优势还有很多，之前的项目上都一直在使用Jenkins，对于大多项目来说是完全满足条件的。这里以Jenkins 2.x为例演示如何安装、配置和使用Jenkins。
下图摘自官网，展示了持续交付场景下应用示例一般流程。
![](/assets/jenkins-by-step/pipeline_flow.png)

## Jenkins Installation
Jenkins由Java编写的开源产品，支持跨平台，安装也非常方便，可以到[Jenkins官网](https://jenkins.io/)下载需要的版本并安装。
![](/assets/jenkins-by-step/download.png)

以下大致列举出几种常用的安装和使用方式，在安装之前请确保已经安装了Java运行环境。

#### War包安装方式
可以直接下载自带Jetty的`*.war`包并运行，非常方便，比较推荐使用，Jenkins默认会运行在`8080`端口。
```
curl -O http://ftp.tsukuba.wide.ad.jp/software/jenkins/war-stable/2.7.1/jenkins.war
java -jar jenkins.war
```

也可以下载最新版本的[jenkins.war](http://mirrors.jenkins-ci.org/war-stable/latest/jenkins.war)。

#### Docker镜像方式
可以直接下载Docker镜像来使用，无需进行安装，但需要有docker环境。
```
docker pull jenkinsci/jenkins
docker run -p 8080:8080 -p 50000:50000 -v /your/home:/var/jenkins_home jenkins
```

其中`/your/home`需要替换为你的主机路径，用于保存Jenkins的Workspace数据，更多细节可以参阅[Github jenkinsci](https://github.com/jenkinsci/docker)。

#### Ansible安装方式
另外，还可以通过Ansible自动化脚本安装，在另一篇博客[Ansible实践篇](/ansible-practice)中也有涉及。首先需要下载jenkins role到本地，假设下载到了`/usr/local/etc/ansible/roles/`目录下。
```
ansible-galaxy install geerlingguy.jenkins
```

然后编写`playbook`来实现**自动化批量安装**，假设创建一个playbook名为`setup_jenkins.yml`，其中`ci-server`是`inventory`文件中的Group或Host。
```
---
- hosts: ci-server
  become_method: sudo
  become: yes
  roles:
    - /usr/local/etc/ansible/roles/geerlingguy.jenkins
```

最后运行命令执行安装，稍等片刻后可访问主机的8080端口：
```
ansible-playbook -i inventory setup_jenkins.yml
```

如果需要设置Java版本可以在安装之前修改`geerlingguy.java`中默认的`java_packages`。更多说明可以参见[ansible-role-jenkins](https://github.com/geerlingguy/ansible-role-jenkins)和[ansible-role-java](https://github.com/geerlingguy/ansible-role-java)。

除了上述安装方式外，也可根据具体的操作系统进行安装，以下给出常用的操作系统下的安装方式。

#### 在RedHat/CentOS中安装
通常有两种安装方式：1. 添加Package Repository后使用yum进行安装和升级；2. 直接下载`*.rpm`进行安装。

**方式一：添加Package仓库安装**
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import http://pkg.jenkins.io/redhat-stable/jenkins.io.key
yum install jenkins
```

**方式二：下载`*.rpm`进行安装**
目前最新版本是`2.7.1-1.1.noarch`，若需要指定安装版本，可到[http://pkg.jenkins-ci.org/redhat-stable/](http://pkg.jenkins-ci.org/redhat-stable/)查看并下载安装。
```
curl -O http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.7.1-1.1.noarch.rpm
sudo rpm -ivh jenkins-2.7.1-1.1.noarch.rpm
```

#### 在Ubuntu/Debian中安装
通常也有两种安装方式：1. 添加Debian Package Repository进行安装和升级；2. 通过直接下载`*.deb`进行安装。

**方式一：添加Package仓库安装**
```
wget -q -O - http://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
deb http://pkg.jenkins.io/debian-stable binary/
sudo apt-get update
sudo apt-get install jenkins
```

**方式二：下载`*.deb`进行安装**
目前最新版本是`2.7.1_all`，也可以到[http://pkg.jenkins-ci.org/debian-stable/](http://pkg.jenkins-ci.org/debian-stable/)查看并下载指定版本。
```
curl -O http://pkg.jenkins-ci.org/debian-stable/binary/jenkins_2.7.1_all.deb
sudo dpkg -i jenkins_2.7.1_all.deb
```

#### 在Mac或Windows下安装
对于Mac OS和Windows，直接下载对应安装包根据提示安装即可：[Mac OS安装包](https://jenkins.io/content/thank-you-downloading-os-x-installer/#stable)、[Windows安装包](https://jenkins.io/content/thank-you-downloading-windows-installer/#stable)。

## Jenkins Setup
当安装完Jenkins 2.x后，访问`http://localhost:8080`默认会进入到登录页面。输入默认用户名`admin`和密码`admin`。
![](/assets/jenkins-by-step/login.png)

登录成功后，Jenkins首先会提示安装推荐插件或自选插件，直接点击`Install suggested plugins`安装默认推荐的插件即可，当然随后也可以在Plugin管理中再选择安装需要的插件。
![](/assets/jenkins-by-step/customize_jenkins.png)

等待常用的一些插件安装完成后就可以开始创建自己的Job了。
![](/assets/jenkins-by-step/suggested_plugins.png)

## Jenkins Jobs
Jenkins Job是很重要的概念，定义了在什么样的情况下执行什么样的任务，以及执行后的操作。

#### 创建一个Job
首先来创建一个Jenkins的Job，点击`create new jobs`或`New Item`来创建一个Job。
![](/assets/jenkins-by-step/home.png)

然后在`Enter an item name`下输入Job的名称，比如：“melon-build”，并选择`Freestyle project`，最后点击OK保存。
![](/assets/jenkins-by-step/new_job.png)

#### Source Code Management 源代码管理
可以指定下载源代码的仓库路径，目前Git是最为流行的VCS，指定Repositories URL，这里以Github托管的项目为例，拉取的Branch为`*/master`。这里的项目仓库为公开仓库，因此直接用HTTP方式即可，如果是私有项目需要添加授权信息。另外特别注意，需要确保Server安装了Git，否则在pull代码时会因找不到命令而失败。
![](/assets/jenkins-by-step/job_source_code.png)

#### Build Triggers 构建触发器
构建触发条件Jenkins提供了多种方式，根据项目需要可以设置不同的触发方式，通常采用`Poll SCM`方式，通过设置Schedule来控制触发条件，Schedule采用的是基于**Cron**语法，但Jenkins对其进行了略微调整，比如设置`H/5 * * * *`表示每5分钟检查一次代码仓库是否有新的code changes，如果有就pull并执行tasks。可以通过点击问号![](/assets/jenkins-by-step/help.png)按钮来获得更多帮助信息，或参考[Cron Wiki](https://en.wikipedia.org/wiki/Cron)和[CronTrigger Tutorial](http://www.quartz-scheduler.org/documentation/quartz-2.x/tutorials/crontrigger.html)。
![](/assets/jenkins-by-step/job_trigger.png)

#### Build 构建和任务
假设项目构建工具采用的是目前相对比较流行的开源自动化构建工具[Gradle](https://gradle.org/)，选择`Use Gradle Wrapper`，添加项目执行的Tasks即可，图中示例最终将执行`gradlew clean build`操作。需要注意的是，如果build依赖特定版本的运行环境，请确保在build机器上安装了对应版本的运行环境，如Java 8等。
![](/assets/jenkins-by-step/job_build.png)

当然，除了Gradle外还可以选择Maven、Ant等方式。当然，如果有一些复杂的自动化工作也可以选择Shell脚本完成，根据项目需要定义。
![](/assets/jenkins-by-step/add_build_step.png)

#### Post-build Actions 构建后置行为
Post-build Actions定义了在完成当前Job的Build任务后接下来需要执行的一系列操作的关系。比如设置在正常完成Build后，进一步获取测试报告和Artifacts，发送Email通知，或并行/串行地触发后续Downstream的一个或多个Jobs，以及部署应用到服务器等。Post-build actions有很多种类型和触发条件，通过下拉列表可以选择，如下图：
![](/assets/jenkins-by-step/add_post_action.png)

其中`Build other projects`表示将自动触发后续Job，`Build other projects(manual step)`表示定义了后续Job，但需要手动点击按钮触发，通常针对部署到High Environments的Job。另外还有一个`Trigger parameterized build on other projects`选项定义了同时触发后续的多个Jobs，比如在build完成后同时触发Integration Test、Acceptance Test以及Sonar等。Jenkins也提供了Deployment相关的插件，总之，Jenkins的插件生态系统管理得很好，需要的功能都可以通过Plugins实现。

下图中定义`melon-build`完成后并行地执行`integration-test`、`acceptance-test`和`sonar`，执行顺序的关系可以被配置在Pipeline View中以可视化的方式展现出来，稍候会在Pipeline View中提及。
![](/assets/jenkins-by-step/job_post_action.png)

至此，针对第一个melon-build的Job设置完成，可以点击`Save`或`Apply`保存了。

#### Custom Workspace 自定义工作区
另外，如果当前Job要重用已经有的Workspace代码，可以选择Tab页`General`->`Advanced`->`Use custom workspace`，然后填写`Directory`，比如填写为`jobs/melon-build/workspace/`。
![](/assets/jenkins-by-step/custom_workspace.png)

#### Test Report 测试报告
另外，针对测试报告，若基于Jacoco，可直接选择`Record JaCoCo coverage report`，当build完成后可自动生成报告。也可以尝试配置`Publish JUnit test result report`中的`Test report XMLs`。
![](/assets/jenkins-by-step/jacoco_report.png)

#### Deployment 部署
部署有多种方式，可以通过`Build`中执行部署脚本，或者在`Post-build actions`中选择相应的Step，比如针对War包部署可以选择`Deploy war/ear to a container`。
![](/assets/jenkins-by-step/deploy_tomcat.png)

## Jenkins View
Jenkins提供了多种视图，如Pipeline View、List View、My View等，目的是为了更好地归类和展示所关注的信息，通常会创建Pipeline View来增强Pipeline可视化效果。首先在Jenkins主页点击Dashboad标题栏最右边的`+`号，然后输入View Name并选择`Build Pipeline View`。
![](/assets/jenkins-by-step/new_view.png)

然后配置Pipeline View，特别注意需要在Layout中选择Initial Job，并且该Job已经配置好Downstream Jobs，然后设定显示的Builds数量和刷新频率等。
![](/assets/jenkins-by-step/config_pipeline_view.png)

配置完成后保存，可以到刚创建的View中查看，可以根据项目需要定义Pipeline Flow。
![](/assets/jenkins-by-step/pipeline_view.png)

## Manage Jenkins 管理

##### Jenkins Plugins 插件管理
Jenkins的插件生态系统管理得很好，通常需要在Workspace中安装很多的插件来实现需要的功能。可以通过`Manage Jenkins`->`Manage Plugins`进入到插件管理页面，可以执行安装、升级、删除插件等操作。
![](/assets/jenkins-by-step/plugins_manager.png)

以下补充罗列一些常用的插件：
- [Gradle plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [Git plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [Github plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [SSH plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [Pipeline plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [Deployment plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [JaCoCo plugin](https://wiki.jenkins-ci.org/display/JENKINS/Plugins)
- [Authorize Project plugin](https://wiki.jenkins-ci.org/display/JENKINS/Authorize+Project+plugin)
- [Checkstyle Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Checkstyle+Plugin)

#### Manage Nodes 管理节点
I will talk about Master and Slave later.

#### Jenkins CLI 命令行模式
Jenkins提供了一个内置的命令行接口，允许通过一些脚本命令远程访问Jenkins功能，这对于自动化的任务和批量操作等非常有利。

首先需要下载`jenkins-cli.jar`，假设你的Jenkins地址是`jenkins.xxx.net:8080`，可以访问以下地址下载：
```
http://jenkins.xxx.com:8080/jnlpJars/jenkins-cli.jar
```

然后可以通过命令行查看帮助，命令为：
```
java -jar jenkins-cli.jar -s http://jenkins.xxx.net:8080/ help
```

当然也可以通过界面查看每个命令的使用帮助，在`Manage Jenkins`->`Jenkins CLI`页面查看到所有Available的命令。
![](/assets/jenkins-by-step/jenkins_cli.png)

如何需要了解更多，可以参考[Jenkins CLI Wiki](https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+CLI)。

#### 其他配置管理
Jenkins除了对插件和节点进行管理，还有系统管理、安全管理、权限配置、命令行工具、用户管理等。在主页选择`Manage Jenkins`进入到Jenkins管理页面，可以选择相应的功能进行配置，每项功能进入后都会有相关的说明，相对也比较简单易懂，这里就不再一一列举了。
![](/assets/jenkins-by-step/manage_jenkins.png)

## Pipeline as Code
Jenkins 2.x推出了`Jenkinsfile`来实现将pipeline定义为代码形式目标，即Pipeline as Code，特别是在集群管理时提升了效率，但会存在一个缺点，问题在于这可能造成在GUI上进行了修改而未修改Jenkinsfile的不一致性，而且无法追踪到这样的修改，所以，如果没有特别的需求，请谨慎选择使用。有兴趣的同学可以研究一下，另外Jenkins CLI也可以尝试一下

## The End
总得来说，Jenkins在常规的项目中使用是不错的选择，强烈推荐使用。另外，后续有时间会写一些关于GoCD，TravisCI以及ConcourseCI的简单使用教程，敬请期待。

----
References

* [Jenkins Home](https://jenkins.io/)
* [Jenkins Tutorial](http://www.tutorialspoint.com/jenkins/index.htm)
* [Step by step guide to set up master and slave machines](https://wiki.jenkins-ci.org/display/JENKINS/Step+by+step+guide+to+set+up+master+and+slave+machines)
* [Build Pipeline Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Build+Pipeline+Plugin)


