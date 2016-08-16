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

## Git准备
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
git config --global user.name "your_name"  # 若无参数，则表示查看当前配置
git config --global user.email "your_email"
```

加上`--global`参数表明所有Project都使用这一配置，如果需要针对某个单独Project进行设置，可以在指定的Project中输入上述命令，但不需要`--global`参数。

#### Repository
现在可以在本地创建一个Git仓库并进行版本管理，最最基础的常用命令如下：
```
git init  # 初始化一个版本库，会生成.git文件夹，可以添加.gitignore文件来标识需要忽略的项

git status  # 查看状态，该命令会非常频繁地用到，建议每一步都check一下状态，确保操作都正确了

git add xxx  # 添加指定的文件到暂存区，可以用Regex匹配，如'*.txt'，对于新增文件可以直接指定
git add .  # 添加当前目录下所有新增或修改的文件，可能会无意加入其他不想加入的文件
git add -u  # 添加已版本管理并修改了的文件，新增文件不会加入，相对保险，建议使用

git commit -m "here is the comment"  # 提交并加入必要的注释说明

git pull origin master  # 从远端仓库拉取master代码，需要设置origin
git push origin master  # 将代码Push到远端仓库的master
```

## Git初阶
除了熟练掌握上述的一些基本命令外，还不能顺畅地使用Git，还需要对更多的命令进行掌握才能达到流畅使用Git的程度，接下来会分别介绍一些常用的命令及其常用参数。

#### 远程 remote
在初始化新项目可能需要添加远程库链接或现在项目的远程库链接时，[Remote](https://git-scm.com/docs/git-remote)命令相当有用，特别是`set-url`还是会常用到的。
```
git remote -v  # --verbose 查看当前已添加的远程库地址
git remote show  # 查看已添加了哪些远程库源
git remote show origin  # 查看指定远程库源origin的详细信息

git remote add origin https://github.com/xxx/demo.git  # 添加origin并指定其远程库地址，通常默认为origin
git remote set-url origin git@github.com:xxx/demo.git  # 修改origin，使用HTTPS或SSH，取决于你的权限
```

当然还有更多的用法和参数，但以上命令足够应对平时使用了，更多特殊场景用途的介绍会放在后续章节讲解。

#### 拉取 pull
在平常的Git使用过程中，[Pull](https://git-scm.com/docs/git-pull)命令使用相当频繁，用于从远端仓库拉取代码，其实是包括了两个命令：[Fetch](https://git-scm.com/docs/git-fetch)和[Merge](https://git-scm.com/docs/git-merge)。
```
git pull origin
git pull -r  # --rebase 把当前分支衍合到upstream的顶端，使得Network保持一条线，更加清晰直接
```

#### 推送 push
另一个常用的命令就是[Push](https://git-scm.com/docs/git-push)，用于把本地的提交推送到远端仓库。
```
git push -u origin master  # 首次Push时需要加-u参数

git push --set-upstream origin master  # 建立upstream与当前分支master的关联关系

git push  # 建立关联关系后默认会推送到对应分支
```

#### 对比 diff
```
git diff HEAD  # 对比修改前后内容，也可以指定特定文件
git diff --staged  # 对比已经加入暂存区的文件
```

#### 分支 branch
通常需要查看、创建、修改或删除一个分支[Branch](https://git-scm.com/docs/git-branch)时需要用到，假设示例中使用的分支名为`feature/card1`。
```
git branch -a  # --all 列出所有分支
git branch --list <pattern>  # 列出符合条件的分支
git branch -r  # --remotes 列出远程分支
git branch -vv  # 查看分支情况，显示sha1, commit和upstream分支信息

git branch feature/card1  # 创建一个新分支
git checkout feature/card1  # 切换到该分支
git checkout -b feature/card1  # 创建并切换到该分支

git branch -m feature/renamed  # --move 修改所在分支的名字
git branch -M feature/renamed  # --move --force 强制修改所在分支的名字

git branch -d feature/card1  # --delete 删除本地分支，不能删除当前所在分支
git branch -D feature/card1  # --delete --force 强制删除本地分支，不能删除当前所在分支
git branch -dr origin/feature/card1  # --delete --remotes 删除远程仓库分支Tracking
git push origin :feature/card1  # 把空分支push到远端，相当于删除远程仓库分支

git branch --set-upstream-to=origin/<branch> master  # 把远程分支和本地分支关联起来
git branch --unset-upstream [<branchname>]  # 移除关联的上游分支，默认针对当前分支

git branch --merged  # 列出已经合并到当前分支的所有分支
git branch --no-merged  # 列出未合并到当前分支的所有分支
```

#### 检出 checkout
checkout
[Checkout](https://git-scm.com/docs/git-checkout)
```
git checkout -- <target>
```

#### 合并 merge


#### 暂存 stash
```
git stash  # 把当前的工作隐藏起来 等以后恢复现场后继续工作
git stash list  # 查看所有被隐藏的文件列表
git stash apply  # 恢复被隐藏的文件，但是内容不删除
git stash drop  # 删除文件
git stash pop  # 恢复文件的同时 也删除文件
```

#### 移除 rm
```
git rm --cache
```

#### 提交记录 log
```
git log -5  # 查看最近5条历史提交记录
git log --pretty=oneline  # 以单行的格式显示提交记录

显示Network的命令
```

#### 操作记录 reflog
```
git reflog  # 查看所有操作记录，该命令非常有用，可以在用于任何操作步骤Hash
```

#### 还原 revert
```
git revert
```

#### 重置 reset
```
git reset --hard HEAD^
```

可以花15分钟在[Try Git](https://try.github.io/)进行简单学习，另外，[Learn Git Branching](http://learngitbranching.js.org/)提供了交互式动画教学和动手实践结合的学习方式，有兴趣可以学习下，同时，也推荐一篇[手把手教你用Git](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=201723758&idx=1&sn=e5b7c27caec76992c348bf30e4bd30e8&scene=2&from=timeline&isappinstalled=0)，练习完成一系列教程后基本就可以流畅地使用Git的常用功能了。

[Bitbucket](https://bitbucket.org/)
[GitHub](http://github.com/)

## Git进阶

git remote show origin

git remote 同时指定多个源

git branch --unset-upstream

#### rebase

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

#### 数据恢复
https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%BB%B4%E6%8A%A4%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%81%A2%E5%A4%8D#_data_recovery
#### 移除对象

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
