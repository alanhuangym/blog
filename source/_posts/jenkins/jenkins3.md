---
title: Jenkins学习笔记3-重启Node.js服务
date: 2017/12/20
tags: 
- jenkins
categories:
- jenkins
---

我们在之前的文章已经完成了当github上python代码有新的push操作之后，jenkins会自动pull最新的代码，并进行编译执行。因为python代码不能体现服务的持续展现，所以我们现在选择一个网站的node.js页面进行尝试。

### 1.项目代码简介





### 拓展

对于网页项目，还有许多的流程可以持续发布。

##### 1.可以部署多个浏览器环境进行测试：

在网站的上线之前，可以部署一些浏览器测试环境，例如Firefox,IE,Safari等等的不同环境，当全部环境通过测试之后，才发布，提高了产品的容错性。