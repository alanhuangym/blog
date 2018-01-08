---
title: Jenkins学习笔记8-Python服务的完整持续部署流程
date: 2018/1/5
tags: 
- jenkins
categories:
- jenkins
---

现在我们已经完成了整个流程的学习，现在我们应用一个简单的python服务，来测试一下效果。

### 1. 整体流程检视

1. 搭建好相应的环境：

   gitlab服务器、jenkins服务器、docker服务器（和相应的redis服务和数据库服务）





业务流程如下：

![](http://ondsf10qe.bkt.clouddn.com/uml1.pdf)

Reference:

\[1\] [Jenkins使用简易教程](http://www.jianshu.com/p/b524b151d35f)

\[2\] https://testerhome.com/topics/10003

\[3\] https://personal-notes.me/%E5%9F%BA%E4%BA%8E-jenkinsdocker-%E6%90%AD%E5%BB%BA%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E4%BA%A4%E4%BB%98%E6%96%B9%E6%A1%88/

\[4\] https://yq.aliyun.com/articles/80459

