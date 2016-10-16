---
title: AWS EC2入门篇
date: 2016-04-11 12:00:08
category: Platforms
tags: [DevOps, AWS EC2, IaaS, 云服务, SSH]
description: Amazon EC2是一个IaaS云服务，主要提供弹性的计算资源，EC2是整个AWS最核心的组成部分，可以在很短的时间内创建、启动和运行不同的类型和大小的EC2实例。
---

### 基本介绍
#### 什么是Amazon EC2

> Amazon Elastic Compute Cloud (Amazon EC2) provides scalable computing capacity in the Amazon Web Services (AWS) cloud.

Amazon EC2是一个IaaS云服务，主要提供弹性的计算资源，通俗地讲，就是提供多种类型的虚拟机。EC2也是整个AWS最核心的组成部分，AWS中有许多的服务需要依赖它。在EC2环境中，虚拟机被称为实例，实例的镜像被称为AMI(Amazon Machine Image)。使用AWS EC2有如下优势：

- 可避免前期的硬件投入，因此能够快速开发和部署应用程序
- 可根据自身需要快速启动任意数量的虚拟服务器、配置安全和网络以及管理存储
- 允许根据需要进行缩放以应对需求变化或流行高峰，降低流量预测需求
- 主要是根据类型和使用时间收费，即使用多少收多少的费用

#### 相关概念和名词解释

Instance: 实例，在EC2环境中，虚拟计算环境被称为实例。
AMI: Amazon Machine Image，亚马逊系统映像，即实例的预配置模板，其中包含服务器需要的程序包（包括操作系统和其他软件）。

IaaS: Infrastructure as a Service, 基础设施即服务。消费者通过Internet可以从完善的计算机基础设施获得服务，这类服务称为基础设施即服务(IaaS)，基于Internet的服务（如存储和数据库）是IaaS的一部分。

IaaS通常分为三种用法：公有云、私有云和混合云。Amazon EC2在基础设施云中使用公共服务器池(公有云)，更加私有化的服务会使用企业内部数据中心的一组公用或私有服务器池(私有云)，如果在企业数据中心环境中开发软件，那么这两种类型公有云、私有云都能使用(混合云)。

Internet上其他类型的服务包括平台即服务(Platform as a Service, PaaS)和软件即服务(Software as a Service, SaaS)。PaaS提供了用户可以访问的完整或部分的应用程序开发，SaaS则提供了完整的可直接使用的应用程序，比如通过 Internet管理企业资源。

### 注册并创建EC2实例
若没有帐号可进入[AWS主页](http://aws.amazon.com/)选择`Create an AWS Account`注册:
![](/assets/aws-ec2-basic/aws_homepage.png)

大概需要填写用户名密码,联系人信息,信用卡信息等，信用卡会被扣掉1美元:
![](/assets/aws-ec2-basic/register_in_process.png)

然后进入AWS控制台选择EC2:
![](/assets/aws-ec2-basic/aws_overview.png)

为了选择最近的地区，可以在[CloudPing](http://www.cloudping.info/)上测试一下Ping速度，选择最快的Singapore:
![](/assets/aws-ec2-basic/choose_location_area.png)
![](/assets/aws-ec2-basic/cloud_ping.png)

点击`Launch Instance`创建一个实例，可以选择`Community AMIs`进行筛选，也可能直接选择Amazon的Linux AMI，据说是速度和性能都进行过优化:
![](/assets/aws-ec2-basic/launch_choose_ami.png)

一定只选择标记为`Free tier eligible`的免费类型，否则运行一段时间就等着哭吧:
![](/assets/aws-ec2-basic/launch_choose_instance_type.png)

根据步骤和提示一步步完成即可，最后启动会选择Key Pair。当系统提示提供密钥时，选择Choose an existing key pair，然后选择已创建的密钥对。另外，也可以新建密钥对，选择Create a new key pair，输入密钥对的名称，然后选择Download Key Pair。这是保存私有密钥文件的唯一机会，因此务必单击进行下载，将私有密钥文件保存在安全位置。当启动实例时，需要提供密钥对的名称，当每次连接到实例时，需要提供相应的私有密钥。
![](/assets/aws-ec2-basic/launch_select_key_pair.png)

最后就可以看到Instances页面出现了已创建成功的实例。如果需要SSH到实例，可以点击`Instances -> 选择Instance -> Connect`查看，Shell Command如下:
``` bash
ssh -i <key.pem> <username>@<instance-address>
```

更多说明请参见[Amazon EC2 的设置](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/get-set-up-for-amazon-ec2.html)和[在 Linux 实例上管理用户账户](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/managing-users.html)

### Instances Management 实例管理
实例(Instance), 即虚拟计算环境。实例的预配置模板，也称为亚马逊系统映像(AMI)，其中包含服务器需要的程序包(包括操作系统和其他软件)。实例CPU、内存、存储和网络容量的多种配置，也称为实例类型。

在`Instances`栏中可以对实例进行Reboot, Stop, Start, Terminate(永久删除)以及其他的网络,安全,卷等设置。
![](/assets/aws-ec2-basic/instances.png)

### Resource & Tags 资源 & 标签
Amazon EC2提供可创建和使用的不同资源，这些资源中的一部分资源包括映像、实例、卷和快照，在创建某个资源时，该资源会被分配一个唯一资源 ID。可以定义某个值标记某些资源，来帮助组织和识别这些资源，即Tags。

标签(Tag)为了`方便管理实例、映像以及其他Amazon EC2资源`，可通过标签的形式为每个资源分配元数据(Meta Data)。`标签可按各种标准(例如用途、所有者或环境)对AWS资源进行分类`，每个标签都包含定义的一个键和一个可选值，例如下图所示：
![](/assets/aws-ec2-basic/tag_example.png)

### Volumes 卷
卷是一种数据块级存储设备，可以连接到单个EC2实例，可以像使用其他物理硬盘一样使用它。使用Amazon Elastic Block Store(Amazon EBS)的数据的持久性存储卷，也称为Amazon EBS卷，提供了三种卷类型：通用型SSD、Provisioned IOPS和磁介质，各卷类型特点可参见: [Amazon EBS 卷类型](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/EBSVolumeTypes.html)。

除了EBS，还有提供临时性块级存储[实例存储](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/InstanceStorage.html)和存储Internet数据的Amazon Simple Storage Service ([Amazon S3](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AmazonS3.html))

卷存储的架构如下图所示:
![](/assets/aws-ec2-basic/architecture_storage.png)

可以选择`elastic block store -> Volumes -> Create Volume`创建一个新卷:
![](/assets/aws-ec2-basic/create_volume.png)

选择Volumes中的一个条目，通过`Actions或右键 -> Attach Volume`连接到某个Instance上:
![](/assets/aws-ec2-basic/attach_volume.png)

在Instance上通过以下命令实现Mount(挂载)创建的卷:

``` bash
df -h  # 查看已挂载的卷
df -T  # 可以查看挂载卷的类型

sudo fdisk -l  # 查看连接到的卷

# 当确定卷的名称为 /dev/xvdf
sudo mkfs.ext4 /dev/xvdf  # 格式化卷类型为ext4
mkdir /mnt/ebs  # 创建文件夹
sudo mount /dev/xvdf /mnt/ebs  # 挂载卷到ebs
df -T  # 再次查看挂载情况

# 若需要卸载卷
sudo umount /dev/xvdf
```

### Snapshots 快照
每个快照代表一个卷在一个特定时间点的状态。快照属于增量备份，这意味着仅保存设备上在最新快照之后更改的数据块。相对容易理解，此处不再赘述。

### Security Groups 安全组

可以使用安全组来控制实例的访问权限，这些安全组类似于一个传入网络防火墙，可以指定允许访问实例的协议、端口和源IP范围。可以创建多个安全组，并给每个安全组指定不同的规则，然后可以给每个实例分配一个或多个安全组，通过这些规则规则确定允许哪些流量可访问实例。

Security Group安全组架构如图所示:
![](/assets/aws-ec2-basic/architecture_security_group.png)

创建新的安全组，选择`Network & Security -> Security Groups -> Create Security Group`进行创建:
![](/assets/aws-ec2-basic/create_security_group.png)

上图显示下拉列表中，可以根据需要选择如SSH,TCP,UDP等，也可以选择Customer Rule来自定义端口号等，还可指定来源IP范围。

选择Security Groups中一个条目，通过`Actions或右键 -> Edit inbound rules或Edit outbound rules`来添加流量流入和流出限制规则:
![](/assets/aws-ec2-basic/edit_inbound_rules.png)

可以根据实际需要添加规则，如果对安全性没有需求，可以不设置防火墙限制(但不推荐)，可直接选择`type`为`All traffic`，选择`source`为`Anywhere`，这样就允许所有类型和源的流量流入。


### Key Pairs 密钥对
Amazon EC2使用公有密钥密码术加密和解密登录信息。公有密钥密码术使用公有密钥加密某个数据(如一个密码)，然后收件人可以使用私有密钥解密数据，公有和私有密钥被称为密钥对。AWS存储公有密钥，个人在安全位置存储私有密钥。如果经常使用SSH那就比较清楚了。

选择`Network & Security -> Key Pairs -> Import Key Pair`可以导入本机的公钥，当然也可以创建Key Pair:
![](/assets/aws-ec2-basic/create_key_pair.png)


### Placement Groups 置放群组


### Elastic IPs 弹性IP
弹性IP地址是专为动态云计算设计的静态IP地址。`实例在重启后会自动重新分配一个与原实例不同的公有IP地址`，如果应用程序需要一个静态IP地址，可以使用弹性IP地址关联到实例，并且在实例发生故障的情况下能够将该地址映射到另一实例，并能够将 DNS主机名用于所有其他节点间通信，从而屏蔽实例故障。

如下图所示选择`Network & Security -> Elastic IPs -> Allocate New Address`分配一个新的EIP:
![](/assets/aws-ec2-basic/new_elastic_ip.png)

选择Elastic IPs中一个条目，通过`Actions或右键 -> Associate Address`输入需要关联的Instance:
![](/assets/aws-ec2-basic/associate_elastic_ip.png)

此时可以通过EIP访问关联到的Instance了。

`特别注意:` 为确保弹性IP地址的有效使用，如果弹性IP地址未与正在运行的实例关联，或者它已与停止的实例或未连接的网络接口关联，Amazon将强制收取小额的小时费用，每小时是$0.005。当实例正在运行时，无需为与该实例关联的某个弹性IP地址付费。当重新映射弹性IP地址次数一个月内超过了100次将收取$0.10费用。在默认情况下，所有AWS账户最多可拥有5个EIP。

### Network Interfaces 网络接口
您可以创建的虚拟网络，这些网络与其余 AWS 云在逻辑上隔离，并且您可以选择连接到您自己的网络，也称为Virtual Private Cloud(VPC)。

### Load Banlancers 负载均衡
可以跨越多个Amazon EC2实例自动分配应用程序的传入流量。

### Auto Scaling
根据定义的条件自动扩展Amazon EC2容量。


### 参考资料
[1] [Amazon EC2 User Guide for Linux Instances](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html)
