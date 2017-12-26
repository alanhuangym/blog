---
title: Jenkins学习笔记6-搭建gitlabs仓库&DockerWeb管理
date: 2017/12/22
tags: 
- jenkins,git
categories:
- jenkins,git
---

### 1. 搭建gitlab仓库

由于需要实现代码仓库的质量管理操作，所以我们需要搭建一个gitlab的仓库。

> GitLab是一个利用 `Ruby on Rails` 开发的开源应用程序，实现一个自托管的Git项目仓库，可通过Web界面进行访问公开的或者私人项目。 

> 　　GitLab拥有与Github类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

现在有了docker，所以搭建运行环境都比较方便，我们这里直接使用docker进行gitlab的搭建。

使用以下命令拉取所需要的docker镜像

```
docker pull sameersbn/redis
docker pull sameersbn/postgresql
docker pull sameersbn/gitlab
```



### 2. Docker WebGUI管理

1. [shipyard](https://github.com/shipyard/shipyard)国产，但是20天前刚刚宣布停止更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins22.png)

2. [rancher](http://rancher.com/) 硬性条件：需要一台linux服务器做主机，至少2G内存和20G硬盘，较少更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins23.png)

3. [kevana](https://github.com/kevana/ui-for-docker)已停止更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins24.png)

4. [portainer](https://portainer.io/)比较轻量，github显示更新及时

此处选择了portainer。

![](http://ondsf10qe.bkt.clouddn.com/jenkins19.png)







References:

\[1\] [持续集成 by www.abcdocker.com](http://blog.csdn.net/abcdocker/article/category/6638595)

\[2\] https://www.jianshu.com/p/060e7223e211?open_source=weibo_search