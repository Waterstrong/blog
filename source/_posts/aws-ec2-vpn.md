---
title: AWS EC2搭建VPN服务器
date: 2016-04-13 23:30:52
category: DevOps
tags: [AWS, EC2, VPN, PPTP, Linux, CentOS, RedHat, Ubuntu]
description: 主要介绍如何在12个月免费的AWS EC2中搭建基于PPTP协议的VPN服务用于翻越GFW，平时Google搜索一下也很方便，会涉及CentOS, RedHat和Ubuntu操作系统。
---

### 基本介绍

主要介绍如何在12个月免费的AWS EC2中搭建基于PPTP协议的VPN服务用于翻越GFW，平时Google搜索一下也很方便，会涉及CentOS, RedHat和Ubuntu操作系统。虽然自己有用其他翻Wall软件，也有我司的VPN服务帐号，但还是想折腾一下，说不定还可以出售给小白同学。

而且自己搭建VPN有诸多好处:
1. 暂时免费，只要不超出AWS免费流量和时间限制;
2. 速度有保障，可以选择最快地区的AWS EC2进行搭建;
3. 自己的帐号自己管理，流量也自己控制，安全有保障。

AWS EC2已经在另一篇Blog中有相关介绍，有兴趣可以参见[AWS EC2入门篇](/aws-ec2-basic)。

VPN: Virtual Private Network, 虚拟专用网络，是一种远程访问技术，主要功能是在公用网络上建立专用网络，进行加密通讯，支持跨平台。
PPTP: Point to Point Tunneling Protocol, 点对点隧道协议，是PPP协议的基础上的增强型安全协议，支持多协议VPN，默认端口号1723。利于PPTP可以快速搭建自己的VPN，并且在很多的移动设备上也支持PPTP，同时PPTP速度也较快，资源消耗也小。

简单介绍了之后，可以尝试以下步骤在服务器上搭建一个VPN服务。

### Step1 - 安装PPTP
**On Ubuntu 14.04 x64:**

``` shell
sudo su  # 登录服务器后切换到超级管理员
apt-get update -y  # 更新源
apt-get install pptpd -y  # 安装pptpd, 同时会自动安装依赖组件ppp和iptables
```

**On CentOS or Red Hat Linux 6.x x64:**

```
sudo su  # 登录服务器后切换到超级管理员
yum update -y  # 更新源
yum install pptpd -y  # 安装pptpd, 同时会自动安装依赖组件ppp和iptables
```

**如果找不到源，返回`No package pptpd available`，如Amazon AMI Linux，可以采用以下方法解决: **

1. 方法一: 下载rpm包直接安装（推荐）

```
# 针对EL6.x版本:
wget -c http://poptop.sourceforge.net/yum/stable/packages/pptpd-1.4.0-1.el6.x86_64.rpm
# 针对EL7.x版本:
wget -c http://dl.fedoraproject.org/pub/epel/7/x86_64/p/pptpd-1.4.0-2.el7.x86_64.rpm

rpm -ivh pptpd-1.4.0-1.el6.x86_64.rpm  # 安装显示安装进度--install--verbose--hash
```

2. 方法二: 需要添加新的源

```
yum repolist  # 查看yum源列表

# 只针对EL7版本:
yum localinstall http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm -y

yum makecache  # 将服务器上的软件包信息下载到本地缓存, 以提高搜索和安装软件的速度
yum repolist  # 可以再次查看新加入的列表
yum install pptpd -y  # 再次执行安装pptpd

# 可用 yum-config-manager --disable <repoid> 删除源
```


### Step2 - 配置PPTP

编辑`/etc/pptpd.conf`:

```
vim /etc/pptpd.conf
```

搜索`localip`并去掉以下字段前的注释符`#`，保存并退出

```
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```

`localip`表示VPN服务器使用的地址，而`remoteip`表示分配给VPN客户端的地址范围，当然也可以自定义设置范围。


### Step3 - 添加DNS解析

针对`Ubuntu`系统，编辑`/etc/ppp/pptpd-options`：

```
vim /etc/ppp/pptpd-options
```

针对`CentOS/RedHat`，编辑`/etc/ppp/options.pptpd`:

```
vim /etc/ppp/options.pptpd
```

在文件中搜索`ms-dns`，去掉以下字段前的注释符`#`，并修改为以下值后保存退出

```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

以上两条配置指定了使用Google Public DNS服务器。


### Step4 - 添加VPN用户
编辑`/etc/ppp/chap-secrets`，添加用户名和密码条目，格式为`[username] [service] [password] [ip]`：

```
# client	server  	secret		IP addresses
vpnuser01	pptpd   	123456		*
```

其中，用户名和密码可自行设置，服务名应为`pptpd`，IP表明允许登录的ip列表，如果允许所有ip可以设置为`*`。


### Step5 - 开启IPv4转发
为了支持IP数据包的转发，需要开启IPv4转发功能。
编辑`/etc/sysctl.conf`:

```
vim /etc/sysctl.conf
```

搜索并找到以下字段，去掉注释并修改为以下值：

```
net.ipv4.ip_forward = 1
```

`= 1`表明开启了服务器内核支持IP数据包转发功能，允许通过PPTP协议在公有IP和私有IPs之间进行数据包转发。


使得修改生效，需要执行：

```
sysctl -p
```

### Step6 - 创建NAT规则
创建网络地址转换，添加防火墙规则到iptables中，在终端执行以下命令

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE  # 将所有目标IP包转向eth0接口
service iptables save  # 添加规则后重启会失效，所以需要保存，若不支持，可添加到rc.local文件中开机自动设置
# 也可使用命令 iptables-save

service iptables restart  # 重启iptables，貌似针对Ubuntu无效(忽略)
```

pptpd默认监听1723端口，可以通过以下命令查看：

```
netstat -nap | grep pptpd
```

(可选)如果端口没有开启则开启相关端口:

```
iptables -I INPUT -p tcp –dport 1723 -j ACCEPT
iptables -I INPUT -p tcp –dport 47 -j ACCEPT
iptables -I INPUT -p UDP --dport 53 -j ACCEPT
iptables -I INPUT -p gre -j ACCEPT
service iptables save
```


### Step7 - 启动PPTP服务

```
chkconfig pptpd on  # 设置开机启动VPN服务
service pptpd restart  # 重启VPN服务, 当然可以用stop/start来停止/启动服务
service pptpd status  # 查看VPN服务当前运行状态
```
可以看到服务已经启动:
![](/assets/aws-ec2-vpn/linux_pptpd_started.png)

针对Ubuntu，即使运行start命令，但查看status还是显示`pptpd is not running`，可以编辑`/etc/init.d/pptpd`文件，搜索`status`找到该行`status_of_proc "$PIDFILE" "$DAEMON" "$NAME" && exit 0 || exit $?`，并添加`-p`参数：

```
status_of_proc -p "$PIDFILE" "$DAEMON" "$NAME" && exit 0 || exit $?
```

针对VPN日志，在CentOS中，VPN服务器默认会写日志到`/var/log/messages`中; 在Ubuntu中，VPN服务器默认会写日志到`/var/log/syslog`中。

以上步骤完成配置正确后，可以利用自己的终端设备连接到VPN上网了。


### Step8 - 使用VPN服务
#### 在Mac OS X上配置VPN

System Preferences(系统设置) -> Network(网络):
![](/assets/aws-ec2-vpn/mac_system_network.png)

选择左下角的`+`号添加VPN，选择PPTP类型，点击create创建:
![](/assets/aws-ec2-vpn/mac_new_vpn.png)

Advanced(高级) -> Options(选项) -> 勾选Session Options中的所有项 -> OK保存:
![](/assets/aws-ec2-vpn/mac_vpn_options.png)

填写VPN相关服务器地址、用户名、密码等信息 -> 点击Apply应用所有修改 -> 点击connect连接VPN服务:
![](/assets/aws-ec2-vpn/mac_vpn_connected.png)

其中Server Address绑定了子域名`aws.vpn.xxx`，在需要更改服务主机时只需要重定位DNS即可，Client终端配置不需要更改，当然要做负载时也很方便，同时也利用记忆。

#### 在iPhone 6s上配置VPN

首先进入Settings设置，选择VPN项（也可以借助第三方软件，如AnyConnect）:
![](/assets/aws-ec2-vpn/6s_vpn_setting.png)

选择PPTP类型，填写服务器地址、用户名、密码等，然后保存:
![](/assets/aws-ec2-vpn/6s_new_vpn.png)

点击connect连接VPN服务:
![](/assets/aws-ec2-vpn/6s_vpn_connected.png)

进入已连接的VPN查看分配的IP，连接时间等详细信息，当然也可以删除VPN:
![](/assets/aws-ec2-vpn/6s_vpn_detail.png)

测试访问Google，在手机浏览器输入`www.google.com`，使用4G数据流量，连接正常，速度没有明显差异:
![](/assets/aws-ec2-vpn/6s_test_vpn.png)

当然，其他设备上也是类似配置，都是一些基础的操作，也该收工了。OK, Just Enjoy~