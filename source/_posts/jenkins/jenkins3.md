---
title: Jenkins学习笔记3-重启Node.js服务
date: 2017/12/20
tags: 
- jenkins
categories:
- jenkins
---

我们在之前的文章已经完成了当github上python代码有新的push操作之后，jenkins会自动pull最新的代码，并进行编译执行。因为python代码不能体现服务的持续展现，所以我们现在选择一个网页网站的node.js页面进行尝试。

### 1.项目代码简介

简单的网页面显示，没有具体的数据存储操作

### 2.代码Jenkins部署

##### 2.1 安装node.js插件

到jenkins插件管理界面下载[Node.js插件](https://wiki.jenkins.io/display/JENKINS/NodeJS+Plugin)

![](http://ondsf10qe.bkt.clouddn.com/jenkins12.png)

安装完，对jenkins进行重启

##### 2.2 插件配置

进入**Jenkins - Manage Jenkins - Global Tool Configuration**设置页面

找到NodeJS设置，点击**NodeJS installations…**

![](http://ondsf10qe.bkt.clouddn.com/jenkins13.png)

帮这个node.js环境填上一个名字，然后选择需要安装的版本号。

保存，即可。

##### 2.3 NPM install

现在选择Build Now，不管有没有job，系统会先安装node.js环境在workspace里。

然后去到job的配置页面，在build里加入**Execute NodeJS script**操作

![](http://ondsf10qe.bkt.clouddn.com/jenkins14.png)

<font color="red">这里如果把语句放入Node.js脚本执行框，会提示错误。（很奇怪，待研究）</font>

所以我们将语句按顺序，放在script下面的shell脚本里，node.js脚本为空。

执行`npm install`对从远程仓库下载下来的代码进行构建，安装第三方库。

##### 2.4 PM2守护进程

- 尝试一

  直接在shell脚本中执行`node index.js`操作，node.js服务正常启动，可以正常访问网页，但是在jenkins中，该job是一直处于执行的阶段，如果有新的代码推送上代码仓库，虽然jenkins可以获取到新的代码，但是因为旧的job一直执行，造成阻塞，只能人工进行停止服务，不能实现代码构建部署自动化

- 尝试二

  因为直接启动node.js不行，所以我们考虑使用nohup进行后台执行，使用脚本`nohup node index.js &`，虽然显示无错误，该job也正常执行完成，但是无法访问网站，查看进程发现不存在node.js进程

- 尝试三

  因为nohup不能持久地运行node.js服务，所以我们选了一个进程守护程序来守护进程，我们选择的是[pm2](https://github.com/Unitech/pm2)，先在shell脚本中加入代码`npm install pm2 -g`，然后再启动进程`pm2 start index.js`，经过测试node.js进程成功常驻后台，网页访问正常，jenkins正常完成job的构建。

  由于我们希望达到自动化代码构建部署的效果，所以jenkins的shell脚本更改为：

  ```
  pm2 stop index.js
  pm2 start index.js
  ```

  这样当代码有更新的时候，先对原先的node.js进行停止，然后重新开启新的node.js服务。

至此，完成node.js代码的自动化构建部署。



### 拓展

对于网页项目，还有许多的流程持续发布，都需要经过浏览器测试的环节。

##### 1.可以部署多个浏览器环境进行测试：

![](http://ondsf10qe.bkt.clouddn.com/jenkins11.png)

在网站的上线之前，可以部署一些浏览器测试环境，例如Firefox,IE,Safari等等的不同环境，当全部环境通过测试之后，才发布，提高了产品的容错性。