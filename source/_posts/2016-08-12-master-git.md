---
title: 优雅地使用Git
date: 2016-08-12 22:29:48
category: Tools
tags: [Git, VCS, SCM, GitHub]
description: Git
published: true
---

## 序
前几天遇到一些关于权限和rebase的问题，另外，之前也有被多次问到在某某情况下怎么处理，某某操作是什么命令，某某命令怎么用等等的问题，当然，还有一些其他不常用的Git命令，但真正需要的时候又比较棘手，估计大部分人只会使用一些简单常规并且在正常流程下的命令，鉴于此，觉得还是有必要分享一下。

如果你已经是Git高手了，这篇博客对你意义不大，不过可以下方留言，补充更多的信息。如果你是从来没有使用过Git的，这篇博客也不太适合你，不过我会给出一些简单说明和入门教程的链接，如果你有兴趣学习的话，想必能够帮到你。如果你是Git新用户，或者使用Git有一段时间了，但经常会遇到一些棘手问题不知所措，那本博客应该会对你有一些帮助。

## 基本介绍

> Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Git是一款免费且开源的分布式版本控制系统(DVCS)，Git是由Linux之父Linus Torvalds在2005年创造出来最初用于管理Linux内核代码，解决与其他贡献者协同开发的问题，Git能以非常高效的方式管理各种规模的项目，分布式意味着每个人的电脑都是一个完整的版本控制库，并且工作时都不需要联网，相对于其他版本控制管理系统，优势不只是一个两个数量级，目前算是世界上最先进的分布式版本控制系统。

更多细节和原理介绍可以阅读[Git官网介绍](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)，这里就不再赘述。

## Git初阶
首先必需要提出来的是，强烈建议使用CLI，不要总想着GUI，如果你能对你执行的操作有完全掌控，你不必担心出现一些非意料的问题，并且当你使用熟练后你会发现CLI比GUI效率高很多，另外，当你从鼠标转移到了键盘上后，你才会感受到，原来生活可以变得如此美好。接下来会简单列举入门的步骤：
#### Installation
请根据操作系统下载并安装Git，请参见[Git Downloads](https://git-scm.com/downloads)。
```
git --version  # 查看安装版本

git help -a  # 列出所有子命令
git help <command>  # 查看某个子命令的帮助

git help -g  # 列出一些概念引导
git help <concept>  # 查看某个概念引导，比如git help tutorial
```

#### Configuration
因为Git是分布式的，需要在本地配置Git用户名和邮箱作为一个标识。
```
git config --global user.name "your_name"
git config --global user.email "your_email"
```

加上`--global`参数表明所有Project都使用这一配置，如果需要针对某个单独Project进行设置，可以在指定的Project中输入上述命令，但不需要`--global`参数。

#### Repository
现在可以在本地创建一个Git仓库并进行版本管理，一般会常用到以下这些命令：
```
git init  # 初始化一个版本库，会生成.git文件夹

git status  # 查看状态，该命令会非常频繁地用到，建议每一步都check一下状态，确保操作都正确了

git add xxx  # 添加指定的文件到暂存区，可以用Regex匹配，如'*.txt'，对于新增文件可以直接指定
git add .  # 添加当前目录下所有新增或修改的文件，可能会无意加入其他不想加入的文件
git add -u  # 添加已版本管理并修改了的文件，新增文件不会加入，相对保险，建议使用

git commit -m "here is the comment"  # 提交并加入必要的注释说明

git pull origin master  # 从远端仓库拉取master代码，需要设置origin
git push origin master  # 将代码Push到远端仓库的master
```


可以花15分钟在[Try Git](https://try.github.io/)进行简单学习，另外推荐一篇[手把手教你用Git](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=201723758&idx=1&sn=e5b7c27caec76992c348bf30e4bd30e8&scene=2&from=timeline&isappinstalled=0)。

## Git进阶

#### 分支 branch

#### 日志 log/reflog
```
git log -5  # 查看最近5条历史提交记录
git log --pretty=oneline  # 以单行的格式显示提交记录

git reflog  # 查看所有操作记录，该命令非常有用，可以在用于任何操作步骤Hash

显示Network的命令
```

#### 还原 revert

#### 重置 reset
```
git reset --hard HEAD^ 
```

#### 切出 checkout
```
git checkout -- <target>
```

#### 对比 diff
```
git diff HEAD  # 对比修改前后内容，也可以指定特定文件
git diff --staged  # 对比已经加入暂存区的文件
```

#### 移除 rm
```
git rm --cache
```

#### 暂存 stash
```
git stash  # 把当前的工作隐藏起来 等以后恢复现场后继续工作
git stash list  # 查看所有被隐藏的文件列表
git stash apply  # 恢复被隐藏的文件，但是内容不删除
git stash drop  # 删除文件
git stash pop  # 恢复文件的同时 也删除文件
```

#### 远程库 remote
```
git remote -v
git remote add origin xxx
git remote set-url origin xxx
```

#### 拉取 pull
```
git pull -r
git branch --set-upstream-to=origin/<branch> master
```

#### 推送 push
```
git push -u origin master

git push --set-upstream origin master
```

#### rebase

#### 合并 merge


## Git高阶
```
git commit --amend -m "xxx"

git format-patch master xxx

git cherry-pick ...


git tag example
git tag -a v1.0 -m “message goes here"
git tag -ln
git push --tags

git checkout v1.0
git tag -d v1.0  # delete tag
git push origin :refs/tags/v1.0  # delete remote tag
git push origin :tagname
git push --delete origin tagname


Generate a new SSH key:
ssh keygen -t rsa -b 4096 "your_email example com"
```

#### 权限
```
To solve the execute permission for gradlew: git update-index --chmod=+x  gradlew
```


git alias config (but not suggest below) `~/.gitconfig`

```

[alias]
co = checkout
ci = commit
pl = pull
ps = push
s = status
st = stash
b = branch
formatlog = log --pretty=format:\"%h %ad | %s%d [%an]\" --graph --date=short
type = cat-file -t
dump = cat-file -p
```

git rebase -i xxx
git rebase xx
git push -f

git merge 和 rebase 讲解

配置快捷方式
或者Zsh+Iterm2


[更多Git相关文章](http://blog.jobbole.com/tag/git/)


[Git Documentation](https://git-scm.com/doc)

----
References
[Git官网](https://git-scm.com/)
[Git Wiki](https://en.wikipedia.org/wiki/Git)
