---
title: 优雅地使用Git
date: 2016-08-12 22:29:48
category: Tools
tags: [Git, VCS, SCM, GitHub]
thumbnailImage: https://git-scm.com/images/reset/workflow.png
description: Git是一款免费且开源的分布式版本控制系统(DVCS)，Git是由Linux之父Linus Torvalds在2005年创造出来最初用于管理Linux内核代码，解决与其他贡献者协同开发的问题，Git能以非常高效的方式管理各种规模的项目，目前算是世界上最先进的分布式版本控制系统。
published: true
---

## 序
前几天遇到一些关于权限和rebase的问题，另外，之前也有被多次问到在某某情况下怎么处理，某某操作是什么命令，某某命令怎么用等等的问题，当然，还有一些其他不常用的Git命令，但真正需要的时候又比较棘手，估计大部分人只会使用一些简单常规并且在正常流程下的命令，鉴于此，觉得还是有必要分享一下。

如果你已经是Git高手了，这篇博客对你意义不大，不过可以下方留言，补充更多的信息。如果你是从来没有使用过Git的，这篇博客也不太适合你，不过我会给出一些简单说明和入门教程的链接，如果你有兴趣学习的话，想必能够帮到你。如果你是Git新用户，或者使用Git有一段时间了，但经常会遇到一些棘手问题不知所措，那本博客应该会对你有一些帮助。

## 基本介绍

> Git is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Git是一款免费且开源的分布式版本控制系统(DVCS)，Git是由Linux之父Linus Torvalds在2005年创造出来最初用于管理Linux内核代码，解决与其他贡献者协同开发的问题，Git能以非常高效的方式管理各种规模的项目，分布式意味着每个人的电脑都是一个完整的版本控制库，并且工作时都不需要联网，相对于其他版本控制管理系统，优势不只是一个两个数量级，目前算是世界上最先进的分布式版本控制系统。

首先特别需要提一下Git的三种状态: committed(已提交), modified(已修改)和staged(已暂存)，同时也需要理解一下git directory(Git版本库), working directory(工作区)和staging area(暂存区)。
* Git Directory(Repository): Git版本库，有一个`.git`的隐藏目录，即版本库，存放了元数据和数据库文件。
* Working Directory: 工作区，指正在工作的目录，`.git`版本库目录除外。
* Staging Area: 暂存区，在版本库中，一个包含待提交信息或缓存的文件。

通常，Git工作的一般流程为：
1. 在工作区中添加或修改文件
2. 将文件快照添加到暂存区中
3. 将暂存区的文件提交并永久存入版本库
![](/assets/master-git/areas.png)

更多细节和原理介绍可以阅读[Git官网介绍](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)，这里就不再赘述。

## Git准备
首先必需要提出来的是，强烈建议使用CLI，不要总想着GUI，如果你能对你执行的操作有完全掌控，你不必担心出现一些非意料的问题，并且当你使用熟练后你会发现CLI比GUI效率高很多，另外，当你从鼠标转移到了键盘上后，你才会感受到，原来生活可以变得如此美好。接下来会简单列举入门准备的步骤：
#### Installation
请根据操作系统下载并安装Git，请参见[Git Downloads](https://git-scm.com/downloads)。
``` bash
git --version  # 查看安装版本

git help -a  # 列出所有子命令
git help <command>  # 查看某个子命令的帮助

git help -g  # 列出一些概念引导
git help <concept>  # 查看某个概念引导，比如git help tutorial
```

#### Configuration
因为Git是分布式的，需要在本地配置Git用户名和邮箱作为一个标识。
``` bash
git config --global user.name "your_name"  # 若无参数，则表示查看当前配置
git config --global user.email "your_email"
```

加上`--global`参数表明所有Project都使用这一配置，如果需要针对某个单独Project进行设置，可以在指定的Project中输入上述命令，但不需要`--global`参数。

#### Repository
现在可以在本地创建一个Git仓库并进行版本管理，最最基础的常用命令如下：
``` bash
git init  # 初始化一个版本库，会生成.git文件夹，可以添加.gitignore文件来标识需要忽略的项

git clone git@github.com:xxx/demo.git [new_name]  # 克隆远程库，可指定新目录名，默认与远程库相同

git status  # 查看状态，该命令会非常频繁地用到，建议每一步都check一下状态，确保操作都正确了

git add xxx  # 添加指定的文件到暂存区，可以用Regex匹配，如'*.txt'，对于新增文件可以直接指定
git add -u  # 添加已版本管理并修改了的文件，新增文件不会加入，相对保险，建议使用
git add .  # 添加当前目录下所有新增或修改的文件，可能会无意加入其他不想加入的文件
git add -A  # --all 添加全部文件

git commit -m "here is the comment"  # 提交并加入必要的注释说明
git commit --amend  # 重新编辑当前提交的注释信息
git commit --amend -m "new message"  # 覆盖已提交的注释信息

git pull origin master  # 从远端仓库拉取master代码，需要设置origin
git push origin master  # 将代码Push到远端仓库的master
```

#### GitHub + SSH Key
如果希望自己的代码上传到[GitHub](http://github.com/)，可以阅读Github官方Demo教程[GitHub Guides](https://guides.github.com/activities/hello-world/)，当然也可以上传到其它代码托管平台，原理步骤基本相同。另外，如果需要添加SSH Key，可以参考[Generating an SSH key](https://help.github.com/articles/generating-an-ssh-key/)。
``` bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"  # 生成新的SSH Key，需要替换自己的Email

ssh-keygen -f ~/.ssh/id_rsa -p  # 修改已生成私钥文件的密码 

ssh-add ~/.ssh/id_rsa  # 添加已有私钥到ssh-agent中
```

## Git初阶
除了熟练掌握上述的一些基本命令外，还不能顺畅地使用Git，还需要对更多的命令进行掌握才能达到流畅使用Git的程度，除了[init](https://git-scm.com/docs/git-init),[clone](https://git-scm.com/docs/git-clone),[status](https://git-scm.com/docs/git-status),[add](https://git-scm.com/docs/git-add)和[commit](https://git-scm.com/docs/git-commit)命令外，接下来会分别介绍一些其他的常用命令及参数。

#### git config
除了上节提到的使用[git config](https://git-scm.com/docs/git-config)命令来配置`user.name`和`user.email`外，还有一些其他常用参数。
``` bash
git config --global -l  # --list 显示所有的全局配置
git config --global user.name  # 显示全局配置的用户名

git config --global push.default simple  # 配置push的默认方式为simple匹配
git config --global alias.s status  # 配置status的别名为s
```

Git配置有三种级别：
* System: `--system`，一般很少会修改系统级配置，对应的系统配置在`/etc/gitconfig`中。
* Global: `--global`，通常会修改全局配置，对应的全局配置在`~/.gitconfig`或`~/.config/git/config`中。
* Local: `--local`默认值，需要对某个项目单独配置时使用，对应的配置在项目目录下的`.git/config`中。

#### git remote
在初始化新项目时，可能需要添加远程库链接，或是在现有项目中修改远程库链接，或同时指定多个源等，[git remote](https://git-scm.com/docs/git-remote)命令相当有用，特别是`add`和`set-url`还是会常用到的。
``` bash
git remote -v  # --verbose 查看当前已添加的远程库地址
git remote show  # 查看已添加了哪些远程库源
git remote show origin  # 查看指定远程库源origin的详细信息

git remote add origin https://github.com/xxx/demo.git  # 添加origin并指定其远程库地址，通常默认为origin
git remote set-url origin git@github.com:xxx/demo.git  # 修改origin，使用HTTPS或SSH，取决于你的权限
```

当然还有更多的用法和参数，更多特殊场景用途可以参数官方的文档说明，但个人认为以上用法足够应对平时使用了。

#### git pull
在平常的Git使用过程中，[git pull](https://git-scm.com/docs/git-pull)命令使用相当频繁，用于从远端仓库拉取代码，其实是包括了两个命令：[git fetch](https://git-scm.com/docs/git-fetch)和[git merge](https://git-scm.com/docs/git-merge)。
``` bash
git pull origin
git pull -r  # --rebase 把当前分支衍合到upstream的顶端，使得Network保持一条线，更加清晰直接
```

#### git push
另一个常用的命令就是推送[git push](https://git-scm.com/docs/git-push)，用于把本地的提交推送到远端仓库。
``` bash
git push -u origin master  # 首次Push时需要加-u参数

git push --set-upstream origin master  # 建立upstream与当前分支master的关联关系

git push  # 建立关联关系后默认会推送到对应分支

git push -f  # --force, 强制Push
```

#### git diff
Git中对比命令[git diff](https://git-scm.com/docs/git-diff)用于对比提交文件或工作区文件的修改情况，通常在准备提交之前可检查一下修改，防止提交不期望的文件。
``` bash
git diff [<filename>]  # 对比unstaged文件与HEAD的区别，可以指定特定文件
git diff HEAD [<filename>]  # 相对最近commit对比修改前后内容，无论是否staged，可指定特定文件
git diff --staged [<filename>]  # 对比已经加入暂存区的文件与最近commit区别，可指定特定文件
```

通常大家更倾向于使用IDE来对比修改，这一点倒觉得没什么问题，毕竟IDE界面上的对比有更强的视觉感，在某种程度上会减少出错的概率。

#### git branch
通常需要查看、创建、修改或删除一个分支时需要用到[git branch](https://git-scm.com/docs/git-branch)，假设示例中使用的分支名为`feature/card1`。
``` bash
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

#### git checkout
再来介绍一下检出命令[git checkout](https://git-scm.com/docs/git-checkout)，除了对分支切换操作，还可以用于丢弃修改，常用的方式如下：
``` bash
git checkout master  # 切换到master分支
git checkout tag1  # 切换到指定的Tag上

git checkout -b <new_branch>  # 基于当前分支，创建并切换到一个新分支
git checkout --orphan <new_branch>  # 创建一个孤儿分支，没有父节点，属于全新的分支

git checkout -- <target>  # 针对unstaged的文件丢弃其修改
```

#### git merge
通常需要在分支间进行分支合并操作，需要用到[git merge](https://git-scm.com/docs/git-merge)命令，比如把`topic`分支merge到`master`上。
``` bash
git merge topic  # 合并topic分支到当前所在分支
```

当然，也可以指定一些其他参数或是合并策略等，用法非常灵活。在merge时通常会遇到合并冲突问题，如果遇到，在自己熟悉的编辑方式下解决冲突，然后运行添加命令`git add -u`，最后根据提示运行`git rebase --continue`即可，**切记勿再commit**。

#### git rm
删除命令[git rm](https://git-scm.com/docs/git-rm)用于从Git的工作树和索引中删除文件。另外，还有一个类似的[git mv](https://git-scm.com/docs/git-mv)命令，但不太常用，主要作用是改名或移动文件，只作简单了解即可。
``` bash
git rm <target>  # 从工作树中移除对象，Git会记录该操作，相当于rm后再git add
git rm --cached <target>  # 从Git索引管理中移除对象，若需要忽略已提交的文件时应使用此命令删除缓存
```

`--cached`参数相当有用的，通常的使用场景是，当某个文件已经被添加到Git管理，但后来又需要在`.gitignore`中忽略。

#### git log
有时需要查看提交记录日志，可以使用命令[git log](https://git-scm.com/docs/git-log)。
``` bash
git log [<options>]  # 显示提交记录，可以指定文件或regex
git log -5  # 查看最近5条历史提交记录
git log -p  # 按补丁格式显示每个更新之间的差异
git log --stat  # 显示每次提交文件的变更统计
git log --graph  # 显示ASCII字符图形表示的每个提交所在的分支及其衍合情况
git log --graph --oneline --all  # 以更详细的信息展示
git log --pretty=oneline  # 以单行的格式显示提交记录，只显示哈希值和提交注释
git log --decorate[=short|full|auto|no]  # 显示出更多的信息，包括ref name等
```

#### git revert
Git提供了撤销某次操作的命令[git revert](https://git-scm.com/docs/git-revert)，相当于对某次提交的回滚操作，该命令会保留之前的所有提交记录，并把撤销操作当作一次新的提交。
``` bash
git revert HEAD  # HEAD~0 撤销最近一次提交
git revert HEAD^  # HEAD~1 撤销上上次的提交
git revert 6a7c70c  # 撤销该HASH对应的提交

git revert [--edit]  # 撤销操作时需要编程注释，默认参数
git revert --no-edit  # 撤销操作时使用默认的注释
git revert -n  # --no-commit 只在本地撤销，不自动提交，可以用于Revert多个commits
```

**以上就是一些常用命令，需要自己练习进行掌握，可以花15分钟在[Try Git](https://try.github.io/)进行简单学习，另外，[Learn Git Branching](http://learngitbranching.js.org/)提供了交互式动画教学和动手实践结合的学习方式，有兴趣可以学习下，同时，也推荐一篇[《手把手教你用Git》](http://mp.weixin.qq.com/s?__biz=MjM5OTA1MDUyMA==&mid=201723758&idx=1&sn=e5b7c27caec76992c348bf30e4bd30e8&scene=2&from=timeline&isappinstalled=0)，练习完成一系列教程后基本就可以流畅地使用Git的常用功能了。**

## Git进阶
除了掌握常用的命令外，还需要在实践中不断地练习，在真正遇到一些问题并解决后才能提升，踩了坑才能更深有体会，正所谓在实践中学习，在跌倒中成长。

#### 数据恢复
只要在Git管理过的对象几乎总是可以恢复的，即使通过回退到了之前的版本，或者执行了一系列的错误操作，看似某些提交被丢失了，但可以通过查看到操作记录日志，并使用[git reset](https://git-scm.com/docs/git-reset)命令实现回退或恢复。
``` bash
git reset --hard HEAD~3  # 将HEAD指向HEAD~3，回退到HEAD~3的版本，即删除最近三次提交HEAD, HEAD^, HEAD~2
git reset --hard 6a7c70c  # 回退到6a7c70c所在的版本
git reset --hard origin/master  # 将本地版本回退到和远程相同
git reset --hard HEAD  # 回退到最近提交的版本
git reset --hard HEAD^ xxx  # 回退xxx文件到上一个版本
```

除了`--hard`参数外，还有另外两个参数，分别进行解释说明：
* --soft: 暂存区和工作区都不会被改变，被reset的文件会在暂存区等待新的提交
* --mixed: 默认选项，暂存区会被reset，但工作区不受影响
* --hard: 暂存区和工作区都被reset，直接回退到指定提交

为了实现回退或恢复，需要查看日志，除了`git log`命令外，还有[git reflog](https://git-scm.com/docs/git-reflog)命令查看Git的引用日志，即操作记录，该命令非常有用，可以检查丢失提交，或查看操作记录Hash并用于重置及撤销等操作。
``` bash
$ git reflog [--all]  # 查看所有引用日志，获取操作记录

$ git reflog  # 查看到引用日志SHA值
6a7c70c HEAD@{0}: checkout: moving from master to develop
627d359 HEAD@{1}: checkout: moving from testbranch1 to master
6811ed6 HEAD@{2}: checkout: moving from release/v2 to testbranch
6811ed6 HEAD@{3}: rebase finished: returning to refs/heads/release/v2
6811ed6 HEAD@{4}: rebase: checkout 6811ed6f7b0128b293ecf17ad365508620d92116
b632cfd HEAD@{5}: checkout: moving from develop to release/v2
6a7c70c HEAD@{6}: rebase finished: returning to refs/heads/develop
6a7c70c HEAD@{7}: pull -r: checkout 6a7c70c3f852da407980147d710850c1b5150ddc
938f88d HEAD@{8}: checkout: moving from feature/card6 to develop
fe9366d HEAD@{9}: commit: feature card 6

$ git log -g  # 以标准格式显示提交和引用

$ git reset --hard 8a0c223  # 恢复到之前的操作

$ git branch recovery fe9366d  # 创建一个新恢复分支，并且指向某个引用日志记录

$ git fsck --full  # 若无reflog了，可用fsck命令显示所有未被其它对象指向的对象，fsck会检查数据库的完整性

$ git fsck --lost-found  # 文件系统检测丢失更改
```

#### 暂存现场
暂存命令[git stash](https://git-scm.com/docs/git-stash)用于保存工作区的修改现场，以便快速切换到其他工作，再在适当时机恢复之前的工作。`stash@{0}`指最近一次暂存记录，`stash@{1}`指上上次的暂存记录，类似于压栈的操作。
``` bash
git stash  # 把当前的工作暂存起来 等以后恢复现场后继续工作

git stash list  # 查看所有被暂存的文件记录列表
git stash show -p stash@{0}  # 查看最近一次的暂存的内容

git stash apply  # 恢复最近一次被暂存的文件，但不删除该暂存记录与内容
git stash apply stash@{1}  # 指定恢复指定的暂存记录
git stash pop  # Warning，恢复被暂存的文件同时，也删除该暂存文件与记录

git stash drop  # 删除最近一次被暂存的所有文件与记录
git stash drop stash@{1}  # 删除指定的暂存记录
git stash clear  # 清除所有暂存记录列表
```

**特别注意：**在使用`stash pop`时要小心了，万一出现误操作，Pop出来的代码与现有代码出现大量冲突，这会导致本来只需要Pull就可以自动解决所有冲突的情况变得复杂，而且代码一旦被Pop出来就没办法恢复了，就只能慢慢一行行解决冲突吧，只能呵呵达了。

#### 部分提交
通常提交是针对一个文件，要么都提交，要么都不提交，但有时候需要在修改某个文件后，只希望暂时提交其中部分已修改内容，其他修改内容打算后续再提交。针对这种对文件进行了多次修改并希望分别提交的场景，首先需要进行部分修改暂存，然后提交，再暂存其它修改，再提交。
``` bash
$ git add -p xxx  # --patch, 可以暂存并提交部分内容

Stage this hunk [y,n,q,a,d,/,e,?]? s # 输入s表示分割该块为更小块，通常是第一步

Stage this hunk [y,n,q,a,d,/,e,?]? y # 依次处理，输入y表示暂存该块

Stage this hunk [y,n,q,a,d,/,e,?]? n # 依次处理，输入n表示不暂存该块
```

* 输入s表示分割该块为更小块
* 输入y表示暂存该块
* 输入n表示不暂存该块
* 输入e表示手工编辑该块
* 输入a表示暂存当前块以及当前文件的后续块
* 输入d表示退出当前文件，不处理后续块，但会转到下一个文件
* 输入q表示退出，不再处理后续块和文件
* 输入/表示搜索匹配给定Regex的块
* 输入?表示查看帮助信息

#### 分支衍合
分支衍合命令[git rebase](https://git-scm.com/docs/git-rebase)非常实用，能够自动把分支结点衍合到upstream顶端从而保证提交树的整洁。
* 在拉取代码时加上`--rebase`参数总是好的，也推荐使用，即`git pull -r`，这样就会把当前分支衍合到upstream的顶端，使得Network保持一条线，更加清晰直接。

* 当在处理Merge或Patch时，会遇到冲突，当解决完成冲突后，会使用到`rebase`命令。
``` bash
git rebase --continue  # 继续完成之前的操作
git rebase --abort  # 放弃rebase过程
git rebase --skip  # 跳过当前rebase过程
```

* 当需要把多个提交点合并成一个commit时，可以使用`rebase`命令，比如，把`6a7c70c`之前的所有提交合并。
``` bash
$ git rebase -i 6a7c70c  # --interactive, 以交互的方式进行rebase操作
# 通常会保留第一个为pick，其他改为squash，千万不要移除中间的任何一次提交，保存退出

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell

# 如果出现错误提示'Could not execute editor'，可以尝试配置编辑器修复此问题：
$ git config --global core.editor /usr/bin/vim
```

* 当出现了Infinity分支，或者需要简化分支的树，可以使用`rebase`命令，假设在release/v2上需要rebase到develop的6a7c70c点上。
``` bash
git checkout release/v2
git rebase 6a7c70c
git push -f
```

#### 使用标签
使用[git tag](https://git-scm.com/docs/git-tag)命令可以帮助建立一系列的Tags，与Branch用法相似，但相对于分支来说更轻量级，并且很适合在workshop中使用，当然也可以作为轻量级项目管理release版本的策略。
``` bash
git tag  # 显示本地所有标签
git tag v1.0  # 在当前点创建名为v1.0的轻量级标签
git tag v1.0 46a359f  # 在46a359f提交点创建轻量级标签
git tag -a v1.0 -m “message here"  # --annotate, --message, 在当前点创建带注释的新标签
git tag -a v1.2 46a359f  # 在某个commit点添加Tag，会弹出输入注释编辑框

git tag -l ‘v1.*’  # --list 列出匹配模式的标签
git tag -ln  # --list, --num, 显示当前所有的标签
git show v1.0  # 显示标签的详细信息

git push v1.0  # 把指定的标签push到远程库，可指定remote
git push --tags  # 把所有标签push到远程仓库，可指定remote

git checkout v1.0  # 切换到标签v1.0
git checkout -b relese1.0 v1.0  # 为标签v1.0创建一个新的分支release1.0

git tag -d v1.0  # --delete 删除本地指定的标签
git push -d origin v1.0  # --delete 删除origin中的标签
git push origin :refs/tags/v1.0  # 删除远程仓库中的标签
```

#### 修改权限
通常在管理可执行脚本时，执行权限默认不会被Git管理，比如`gradlew`在本地创建完成并且也可以运行，但在服务器上拉取代码后发现没有执行权限，因此会导致执行脚本任务失败，为了解决这一问题，需要用到[git update-index](https://git-scm.com/docs/git-update-index)命令。
``` bash
git update-index --chmod=+x gradlew  # 添加执行权限
git update-index --chmod=-x test.sh  # 移除执行权限
```

#### 补丁技巧
有时会需要在某个分支上针对某个commit打patch，然后再应用到另一个分支上，作为一种补丁的形式出现。首先介绍一下[git cherry-pick](https://git-scm.com/docs/git-cherry-pick)命令，该命令允许从其他分支上选取某一个Commit，再应用到当前分支上。
``` bash
git checkout test_branch  # 首先切到某个需要被补丁的分支
git cherry-pick 46a329f  # 然后应用某个其他分支上的commit
```

当需要生成Patch文件时，可以用命令`git diff HEAD >> xxx.patch`，然后再使用命令`git apply xxx.patch`应用该Patch。

另外，可以参阅更多命令[git format-patch](https://git-scm.com/docs/git-format-patch)和[git apply](https://git-scm.com/docs/git-apply)。

#### 回收垃圾
[git gc](https://git-scm.com/docs/git-gc)和[git count-objects](https://git-scm.com/docs/git-count-objects)命令用于回收垃圾和查看数据库占用空间的。如果有太多松散对象和大文件对象，占用了太多空间，可以尝试手动运行一些命令减少空间占用。
``` bash
git gc  # 压缩文件，回收垃圾，同时查看数据库占用空间
git gc --auto  # 手动执行自动垃圾回收

git count-objects -v  # 查看对象数量和占用空间

git prune --expire now  # 删除过期的文件
```

#### 谁的代码
还有一个很有用的命令[git blame](https://git-scm.com/docs/git-blame)，可以查看到某个文件的每一行修改记录，有时需要确认是谁修改了某一行代码，该命令会很有帮助。
``` bash
git blame xxx  #  查看文件的每一行作者、最新提交和操作时间等
```

#### 高效别名
为了提升Git命令的操作效率，通常会配置Git别名来简化命令输入，其中一种方式是自己在`~/.gitconfig`中配置，但不是首选推荐使用。
``` bash
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

还有一种推荐使用的方式，如果你使用的[Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh)+[iTerm2](http://www.iterm2.com/index.html)的命令行方案，那么你可以感受到`Oh My Zsh`带来的优势了，提供了[git plugin](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git)并设定了统一的更加简短的别名，常用的Alias如下：
``` bash
gst  # git status  ★★★★★

ga  # git add  ★★★★★
gaa  # git add --all

gb  # git branch  ★★★★★
gba  # git branch -a

gl  # git pull  ★★★★★
gup  # git pull --rebase  ★★★
gupv  # git pull --rebase -v

gp  # git push  ★★★★★

gc  # git commit -v  ★★★★★
gc!  # git commit -v --amend  ★★★★★
gcmsg # git commit -m  ★★★
gca  # git commit -v -a
gcam  # git commit -a -m
gca!  # git commit -v -a --amend

gco  # git checkout  ★★★★★
gcm  # git checkout master  ★★★
gcb  # git checkout -b

gd  # git diff  ★★★★★
gdca  # git diff --cached  ★★★

gsta  # git stash  ★★★★★
gstaa  # git stash apply
gstd  # git stash drop
gstl  # git stash list
gstp  # git stash pop
gsts  # git stash show --text

gcf  # git config --list

gcp  # git cherry-pick

glog  # git log --oneline --decorate --color --graph  ★★★★★
```

其中，`★★★★★`表示强烈推荐使用的项，太多Alias会导致记忆混乱，个人觉得记住基本命令的简写即可，参数由自己显示决定，另外，还是需要掌握全称更好，毕竟不是每台电脑的配置都和你的一样，但自己使用确实会方便快捷很多。
若需再定义Alias，可以在`~/.zshrc`中加入自定义的Alias，如：`alias gau='git add -u'`, `alias gs=gst`。

另外，补充一些在PowerShell中定义常用的相同Alias，在PowerShell安装目录下新建`profile.ps1`文件，并写入以下代码：
``` powershell
# gst, git status
function Get-GitStatus { & git status }
New-Alias -Name gst -Value Get-GitStatus

# gb, git branch xxx
function Get-GitBranch([string]$branch) { & git branch $branch}
New-Alias -Name gb -Value Get-GitBranch

# gba, git branch -a
function Get-GitBranchAll { & git branch --all }
New-Alias -Name gba -Value Get-GitBranchAll

# gco, git checkout xxx
function Get-GitCheckout([string]$param) { & git checkout $param }
New-Alias -Name gco -Value Get-GitCheckout

# gau, git add -u
function Get-GitAddUstaged { & git add -u }
New-Alias -Name gau -Value Get-GitAddUstaged

# gaa, git add -a
function Get-GitAddAll { & git add . }
New-Alias -Name gaa -Value Get-GitAddAll

# gcmsg, git commit -m xxx
function Get-GitCommitMsg([string]$message) { & git commit -m $message }
New-Alias -Name gcmsg -Value Get-GitCommitMsg

# gup, git pull -r
function Get-GitPullRebase { & git pull --rebase }
New-Alias -Name gup -Value Get-GitPullRebase

# glog, git log --oneline --decorate --color --graph
function Get-GitLogGraph { & git log --oneline --decorate --color --graph }
New-Alias -Name glog -Value Get-GitLogGraph
```

## 结束语

对于Git初阶部分，都是平常自己使用的一些命令，对于Git进阶部分，除了补充一些命令外，也是自己使用Git的一些心得体会和技巧，无论怎样，只有通过自己亲身实践，不断练习才能真正掌握。以上所有都只是作为一名普通Git用户的实践方式，如果需要开发一些基于Git的应用等，那就需要了解更深层次的底层命令和原理了。

----
References
* [Git官网](https://git-scm.com/)
* [Git Wiki](https://en.wikipedia.org/wiki/Git)
* [Git Documentation](https://git-scm.com/doc)
* [Git Reference](https://git-scm.com/docs)
* [GitHub Help](https://help.github.com/)
* [更多Git相关文章](http://blog.jobbole.com/tag/git/)
