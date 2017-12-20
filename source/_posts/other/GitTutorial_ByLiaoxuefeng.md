---
title: Git Tutorial
date: 2017/3/16
tags: 
- git
categories:
- CUHK_Notes
---



Source:

http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000

# Git Tutorial

Download:

For Mac user, you can simply go to the offical website to download the package and install it.

Link:https://git-scm.com/downloads



在终端里面，使用

pwd

显示当前所在的目录地址

/Users/alan/git/learngit

使用

git init 创建一个新的仓库



修改：

git add 文件名（带后缀）

git commit -m "说明" 



查看本地仓库状态

git status

查看修改在何处

git diff



查看修改日志

git log

美观可以改为

git log —pretty=oneline



返回上一次修改后的版本（每一个^返回一个版本）（HEAD 表示当前版本）

git reset —hard HEAD^



前进版本(hard 后面是版本号，从log中可以得出)

git reset —hard b3fe8e

找版本号可以

git reflog (命令日志)

从这里可以找到版本号



舍弃工作区的修改（返回add状态的版本或者最新commit的版本）

git checkout — readme.txt 

舍弃缓存区的修改

git reset HEAD readme.txt



关联远程仓库(先有本地库再有远程库的时候才需要)

git remote add origin git@github.com:alanhuangym/learngit.git

把本地内容推送到远程仓库上（第一次加-u）

git push -u origin master

git push origin master



从远程库中下载到本地库中(会自动新建文件夹)

git clone git@github.com:alanhuangym/gitskills.git



分支：

查看分支

git branch

创建分支

git branch 分支名

切换分支

git checkout 分支名

在master下面合并分支

git merge 分支名

删除分支

git branch -d 分支名

