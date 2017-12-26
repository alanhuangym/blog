---
title: Jenkins学习笔记1-Jenkins入门&Python代码初试
date: 2017/12/18
tags: 
- jenkins
categories:
- jenkins
---

### 1.敏捷开发

普通的程序开发，是按照瀑布流一样的流程往下走的。

但是时至今日，客户的需求变化速度非常之快，而且生产环境也在不断变化，所以以往的单一的以计划来完成软件开发已经不可取了。

所以我们需要采取敏捷开发，***

##### 1.1 什么是持续集成：

持续集成是指开发者在代码的开发过程中，可以频繁的将代码部署集成到主干，并进程自动化测试 
![image_1b4gk79111oqbf1l16qd80kqqh8q.png-73.4kB](http://static.zybuluo.com/abcdocker/141k7ufqcy5zmgjvzlsgx444/image_1b4gk79111oqbf1l16qd80kqqh8q.png)

##### 1.2 什么是持续交付：

持续交付指的是在持续集成的环境基础之上，将代码部署到预生产环境 
![image_1b4gk7hef1v4n6ino701pruj9n97.png-133.6kB](http://static.zybuluo.com/abcdocker/z47rzhss99x3gxahr93ovn45/image_1b4gk7hef1v4n6ino701pruj9n97.png)

##### 1.3 持续部署：

在持续交付的基础上，把部署到生产环境的过程自动化，持续部署和持续交付的区别就是最终部署到生产环境是自动化的。 
![image_1b4gk7o2t162hb71vcg1bb5p989k.png-132.2kB](http://static.zybuluo.com/abcdocker/4i4r6iwdisn4zmsmhqewxoew/image_1b4gk7o2t162hb71vcg1bb5p989k.png)



### 2.Jenkins入门

##### 2.1 Jenkins安装

Jenkins是Java编写的，所以需要安装JDK。可以去[Oracle官网](https://www.oracle.com/java/index.html)进行下载

在[Jenkins官网](https://jenkins.io/)下载.war文件，然后直接在命令行运行`java -jar jenkins.war`，如果需要后天启动可以`nohup java -jar jenkins.war &`

当命令行出现`INFO: Jenkins is fully up and running`即已经成功启动了服务。

默认端口为8080，可以在启动命令后加上 `—httpPort=8081`更改默认的启动端口。

启动设置还有：

`—httpsPort=$HTTP_PORT`表示使用 https 协议。
`—httpListenAddress=$HTTP_HOST`用来指定 jenkins 监听的 ip 范围，默认为所有的 ip 都可以访问此 jenkins server。

当然也可以把.war包部署进tomcat的webapps里面。

##### 2.2 Jenkins启动

启动了Jenkins服务之后，我们登录`localhost:8080`提示需要输入安全token。

根据提示，安全token在安装完jenkins会在本地的文件夹当中，例如macos会存放在`/Users/Shard/Jenkins/home/secrets/initialAdminPassword`

输入完密码之后，会提示安装建议安装的插件。

![](http://static.zybuluo.com/abcdocker/2dtjyesj3ae5zmyhxmer2kl2/image_1b4gkgcvk1c641d3k1e761jn6h10ae.png)

然后会提示创建一个管理员账户

![](http://static.zybuluo.com/abcdocker/wkz0tuy8b3855c1q3s64vd7x/image_1b4gkkuopf351o7fltt17on13jbl.png)

创建完成则正式进入jenkins。

![](http://static.zybuluo.com/abcdocker/tcz7eoamylmpjl1dekyzt6d0/image_1b4gkl8sf1o1m12b9h9q1p85hsc2.png)

**

macos如果遇到没有权限进入的文件夹，右键get info 然后添加权限用户即可

例如配置jenkins需要进入的`/Users/Shard/Jenkins/home/secrets`

**

##### 2.3 Jenkins的简介

Jenkins的关闭、重启等操作，可以通过查找进程，获取pid号，然后kill进程来完成。

但是更便捷的方法是，在jenkins的url中执行一些命令来操作jenkins，

如下http://[jenkins-server]/[command] 命令可以为：

- `exit` 关闭 jenkins
- `restart` 重启 jenkins
- `reload` 重新载入配置 configuration

Jenkins的目录（例如macos在`/Users/Shared/Jenkins`）中的`Jenkins/Home/jobs`是jenkins的核心内容，包含了jenkins的项目的配置、构建日志等重要信息，对jenkins进行备份，主要备份该文件夹即可。

Jenkins其实是一个开源的平台，主要的功能实现都是依靠插件完成，所以平台的设置没有太多，比较重要的是插件设置，所以我们将对使用到的具体的插件设置在后面讲述。

### 3.简易Python脚本启动

首先我们点击左上角的**New Item**，新建一个job

![](http://ondsf10qe.bkt.clouddn.com/jenkins2.png)

我们需要对这个job命名，因为是一个简单的实验项目，所以我们选择的是**Freestyle project**

其中的**Pipeline**其实是官方推荐的持续部署的写法，但是有其特殊的编写语法，所以我们会在后面再去使用

![](http://ondsf10qe.bkt.clouddn.com/jenkins3.png)

然后我们进入了一个freestyle job的设置页面。

- **Discard old builds**

   一个job由于触发器的触发，会产生许多的构建记录，所以这个选项可以让我们选择是否丢弃过时的构建记录。阈值的设置可以为日期（即丢弃X日前的构建记录）或数量（即只保存最新的X个构建记录）。

- **Github project**

  由[GitHub plugin](http://wiki.jenkins-ci.org/display/JENKINS/Github+Plugin)产生的选项，可以输入该job的github地址，从而可以在后面的trigger触发器中选择使用**GitHub hook trigger for GITScm polling**（当有更改push上github后触发触发器）

- ***This* project is parameterised** 

  为这个job添加一些变量参数，当执行构建或触发触发器而构建的时候，使用到这些参数，从而实现不同的功能，参数的设定可以由默认值，当该项被勾选后，Build Now*会变更为Build with Parameters*，需要键入相应的参数


- **Throttle builds**

  由 [Branch API Plugin](http://wiki.jenkins-ci.org/display/JENKINS/Branch+API+Plugin)产生的选项，对于构建进行一个最小时间间隔的强制设定，例如一天最多构建6次job。但是这个选项**不会**让job在时间段内**平均**构建，例如设置了一天6次，但是不是每隔4个小时就执行一次。

- **Disable this project**

  暂时关闭这个job，从而停止触发器的触发等操作。

- **Execute concurrent builds if necessary**

  勾选了这个选项，这个job并发的构建（例如同时点击数次Build Now）会异步进行构建。而默认非勾选情况下，是阻塞执行构建的。

![](http://ondsf10qe.bkt.clouddn.com/jenkins4.png)

然后就是代码仓库的管理选项了。

这个项目使用的是使用github作为代码仓库，所以我们选择第二个。（如使用SVN则选择第三个Subversion）

其中我们需要键入仓库的地址，对于github或者gitlab，我们需要使用非对称密钥对去进行身份验证，这里使用的是RSA算法生成的密钥。将jenkins所在本机生成的公钥填写入github或gitlab的密钥验证中，从而实现身份验证。

其中也需要选择远程仓库的分支，默认是master分支。

![](http://ondsf10qe.bkt.clouddn.com/jenkins5.png)

接下来是选择构建触发器的选项，当触发器的条件满足时，则对job进行构建

- **Trigger builds remotely (e.g., from scripts)**

  在该选项设定一个token口令，使用脚本对该项目进行立刻构建指令

  ``JENKINS_URL`/job/simple-python/build?token=`TOKEN_NAME` or /buildWithParameters?token=`TOKEN_NAME``

  尾部最好加上`&cause=Cause+Text`对该次构建进行注释，方面翻查操作日志

- **Build after other projects are built**

  可以选择在一个或多个项目完成构建之后触发触发器，进行构建，熟练地使用该触发器，可以形成一个有效的pipeline工作链

- **Build periodically**

  周期性地对项目进行构建，使用的语法是[cron](http://blog.csdn.net/k_scott/article/details/8508290)

  例如 * * * * * 则代表每分钟进行一次构建

- ~~**GitHub hook trigger for GITScm polling**~~

  ~~上文所说到的，当github有更新时，进行构建~~   暂时没研究怎么用，可以使用以下的**Poll SCM**

- **Poll SCM**

  当使用非github作为远程仓库时，使用该触发器，周期性对远程仓库进行检测，当发现有更新时进行构建

  使用的时间安排表语法也是[cron](http://blog.csdn.net/k_scott/article/details/8508290)

![](http://ondsf10qe.bkt.clouddn.com/jenkins6.png)

接下来是一些构建时的，对环境的选项设置

- **Delete workspace before build starts**

  默认是当开始新的构建时，对该job的工作空间进行全部删除，也可以通过设置，只删除某些文件

- **Abort the build if it's stuck**

  可以设置某一个具体的超时时间，当构建阻塞超过超时时间后，对该构建进行中止

- **Add timestamps to the Console Output**

  添加时间戳到控制台输出

- **Use secret text(s) or file(s)**

  对文件或密码进行加密

- **With Ant**

  为Apache Ant搭建环境

![](http://ondsf10qe.bkt.clouddn.com/jenkins7.png)

接下来是根据需要进行构建的脚本编写，这里是测试一个简单的python程序，所以直接使用命令行语句

`python test.py`

其中还有一些例如代码质量管理的**Sonar Gates**会在后述解释

![](http://ondsf10qe.bkt.clouddn.com/jenkins8.png)

最后还可以选择在构建完成之后做其他动作，例如构建其他的项目，形成一个pipeline。

这里没有使用。

![](http://ondsf10qe.bkt.clouddn.com/jenkins9.png)

![](http://ondsf10qe.bkt.clouddn.com/jenkins10.png)

至此，这个job就简单设置完成了，每当检测到远程代码仓库有更新时，进行代码构建，输出到控制台。

这是一个简单的python程序。

### 4. Jenkins的master-slave模型

![](http://images.cnblogs.com/cnblogs_com/itech/build/jenkinsslavetpye.PNG)

Jenkins也有分布式的构建，模式是master-slave的模型，master主管所有的job的运行情况。slave可以设置成不同的生产环境，master分配job到slave中，实现相应的操作。

Reference：

\[1\] [持续集成 by www.abcdocker.com](http://blog.csdn.net/abcdocker/article/category/6638595)

\[2\] [Jenkins入门 by itech](https://files.cnblogs.com/files/itech/Jenkins%E5%85%A5%E9%97%A8.pdf)

\[3\] [Jenkins分布式构建](http://www.yiibai.com/jenkins/jenkins_distributed_builds.html)