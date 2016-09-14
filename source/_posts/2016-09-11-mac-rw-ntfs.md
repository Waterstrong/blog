---
title: Mac如何读写NTFS磁盘
date: 2016-09-11 21:10:13
category: [Techniques]
tags: [Mac, NTFS, 移动硬盘]
description: 如何实现在Mac下读写NTFS磁盘，除了购买Mac读写NTFS磁盘的工具，或使用虚拟机作为媒介再Share给Mac的方式外，还可以使用更为快捷、简单、干净的Mac内置原生方式解决写NTFS硬盘的问题。
published: true
---

其实老早就遇到了Mac下写NTFS的问题，本来因为有两块移动硬盘，把其中一块500G的格式化成了Mac支持的格式，用于备份一些重要数据，而另一块1T之前已经存了不少数据，也懒得麻烦就一直放着没有管，但最近电脑数据太多，已经报警磁盘空间不足，不得不进行数据备份了，最早的硬盘已差不多满了，但又不想改磁盘格式，并且以后还可能在Win下拷贝数据，必须得想办法解决在Mac读写NTFS磁盘问题。

可能也有很多同学遇到和我类似的问题，但不用担心，其实是有办法解决的，可能有些同学愿意购买Mac读写NTFS磁盘的工具，或者使用虚拟机作为媒介再Share给Mac，个人而言，不希望安装过多的软件，有洁癖，虚拟机的方式太慢太啰嗦，怕麻烦，所以只能寻求Mac内置原生方式，而这里就将介绍这种更为快捷、简单、干净的方式，只需要两步即可。
![](/assets/mac-rw-ntfs/disk_utility.png)

#### Step1 配置fstab
在终端输入如下命令编辑`fstab`文件，并写入`LABEL`以及其值，若文件不存在将新建该文件。
``` shell
$ sudo vim /etc/fstab

LABEL=Datum none ntfs rw,auto,nobrowse  # 使用磁盘名方式
LABEL=68D036FA-EE09-4B80-AC93-01E54D51059A none ntfs rw,auto,nobrowse  # 使用UUID方式
```

其中，`rw`表示读写方式，`nobrowse`表示默认不在finder的边栏中显示，如果不加`nobrowse`可能在挂载后还是只读模式。`Datum`是硬盘名或分出来的磁盘名，如果有多个磁盘可以重复再写一行，当然也可以使用磁盘UUID，可在应用程序中打开`Disk Utility`工具查看，或者使用命令行方式查看，比如查看Datum盘信息的命令为：
``` shell
diskutil info /Volumes/Datum
```

#### Step2 打开磁盘
由于设置了默认不在Finder边栏显示，因此需要手动打开磁盘，确保磁盘已经挂载成功，可以通过`Disk Utility`或在命令行挂载并查看。
![](/assets/mac-rw-ntfs/drive_info.png)

打开硬盘磁盘的方式有两种：
- 在Finder中使用快捷键`cmd+shift+g`并输入`/Volumes/Datum`后可进入该磁盘目录
- 命令行中运行`open /Volumes/Datum`后进入该磁盘目录

![](/assets/mac-rw-ntfs/volumes_folder.png)

此时，应该可以对磁盘进行读写操作了，为了下次更加的方便，也可以直接拖拽磁盘到Finder边栏的Favorites中。


**注意：**道听途说该方式有导致磁盘损坏和数据丢失的风险，使用时可能需要留意一下，尽量保存一份副本，到时可不要来找我哦^_^。
