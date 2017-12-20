---
title: Jenkins学习笔记4-使用Docker对环境和代码打包
date: 2017/12/21
tags: 
- jenkins
categories:
- jenkins
---

当我们已经完成了了尝试重启Node.js服务之后，我们应该发现，远程的代码仓库对于node.js的代码，是没有保存Node Modules的。这个文件夹包含的都是node.js需要的第三方的环境库等的包，所以不适合放入远程仓库里。

于是我们每次都需要执行一次`npm install`的操作，确保下载全部的包。但是这样的操作十分耗时耗力，所以我们考虑到使用docker对我们的基础环境进行打包，生成dockerfile和dockerimage，从而就不需要每次都对生产环境进行更新操作。

![](http://ondsf10qe.bkt.clouddn.com/jenkins11.png)

### 1.docker简介



### Swarm集群拓展

