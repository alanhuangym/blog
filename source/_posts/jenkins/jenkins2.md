---
title: Jenkins学习笔记2-Sonar实现代码质量分析（提交到代码仓库后）
date: 2017/12/19
tags: 
- jenkins
categories:
- jenkins
---

### 1.代码质量管理

此次代码质量管理的需求，是在用户将本地的代码提交到远程仓库后，进行代码质量分析。

此处的代码质量不单单指有无拼写错误、格式错误等，还包括代码重复率、鲁棒性、注释率等等的指标。所以单纯的syntax错误已经不能满足我们的需求，我们希望团队能够写出高效的代码。

### 2.SonarQube

Sonar是一个用于代码质量管理的开放平台，同样支持很多的插件，可以集成不同的测试工具，从而对多种语言(包括Java、C/C++，Javascript，C#等)的代码进行分析，并且以数据可视化的界面进行显示，提高了代码的审查率。

##### 2.1 SonarQube安装

可以到[官网](https://www.sonarqube.org/downloads/)下载最新的LTS包，Windows平台的话可以直接执行wrapper.exe文件，即可启动Sonar服务，unix平台的可以通过`./sonar.sh start`启动服务。

`./sonar.sh stop`停止服务

`./sonar.sh restart`重启服务

当出现

```
Starting SonarQube...
Started SonarQube.
```

则启动成功

默认的端口是9000，访问`localhost:9000`需要进行登录。

默认的用户名和密码都是`admin`

##### 2.2 SonarQube Scanner安装

除了SonarQube服务外，我们还需要安装一个SonarQube Scanner作为分析源码的软件。

[下载地址](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner)下载相应系统版本的scanner之后，进行解压缩，放置到适当的位置。

最好能将该文件夹的bin文件夹添加入环境变量。

```
#Macos下添加环境变量的方法
#在命令行输入
$ vi ~/.bash_profile

#在文件的最后添加类似的路径
export PATH=/usr/local/sonar-scanner-3.0.3.778-macosx/bin:$PATH
#即完成了添加
```

然后可以进入对配置文件`./conf/sonar-scanner.properties`进行设置

```
#基础设置两个就可以了
#----- Default SonarQube server 设置SonarQube的服务地址
sonar.host.url=http://localhost:9000

#----- Default source code encoding 设置默认的源码编码格式
sonar.sourceEncoding=UTF-8
```

然后当我们需要对一个项目进行代码分析时，需要在项目的根目录上创建一个`sonar*-project.properties`文件，对项目信息进行配置

```
# 为这个项目配置唯一的标识码（必填）
# 例如以下的URL的id就是标识码
# http://localhost:9000/dashboard?id=myproject
sonar.projectKey=myproject
# 这是在SonarQube上展示的名字和版本号(重要选填)
sonar.projectName=My project
sonar.projectVersion=1.0

# 源码的地址(必填)，有多个路径用','符号隔开
# 不同系统需要注意路径的'/'和'\'符号
sonar.sources=.
 
# 编码，默认是系统编码(选填)
#sonar.sourceEncoding=UTF-8
```

然后直接命令行运行`sonar-scanner`即可，我们现在可以在SonarQube上看到代码质量分析报告了。

如果不是在当前目录运行代码分析，可以设定字段`sonar.projectBaseDir=`来制定代码的位置。

当然如果不希望重新为每一个项目创造一个配置文件，或者无法再根目录创建文件，还有两种替代性的方法可以配置：

- 可以在命令行启动sonar-scanner的时候添加上配置，例如

```
# -D<arg>定义配置
sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=src1
```

- 可以在另外的位置创建配置文件，然后启动sonar-scanner的时候设置路径

```
sonar-scanner -Dproject.settings=../myproject.properties
```

##### 2.3 MySQL配置

sonar内部是有配置数据库的，但是效率低不能应用于生产环境，所以我们需要配置自己的数据库，用户保存sonar的分析报告和配置等。

这里选择的是MySQL的分支MariaDB，因为MySQL有两种引擎，MyISAM和InnoDB，而MyISAM是比较老旧的引擎了，所以sonar不支持MyISAM，所以我们选择使用MariaDB。

配置文件在`sonarqube-6.7/conf/sonar.properties`

```
sonar.jdbc.username=sonar            #数据库用户
sonar.jdbc.password=sonar@pw     #数据库密码

sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&character    Encoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance
```

只需要把数据库的用户名和密码填上，然后解除SQL的注释就可以了。

可能还需要将数据库驱动放入`sonarqube-6.7/extensions/jdbc-driver/mysql`中

##### 2.3 SonarQube 简介

> SonarQube是一个用于代码质量管理的开源平台（Java开发），用于管理源代码的质量，可以从七个维度检测代码质量,通过插件形式，可以支持包括Java，C#，C/C++，PHP，PL/SQL，Cobol，Web，XML，JavaScrip，Groovy等等二十几种编程语言的代码质量管理与检测。



参考资料：

\[1\] [Github Plugin for Sonar](https://docs.sonarqube.org/display/PLUG/GitHub+Plugin)

\[2\] [SonarQube Scanner](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner)

\[3\] [Analyzing with SonarQube Scanner for Jenkins](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+Jenkins)

\[4\] [Analysis Parameters](https://docs.sonarqube.org/display/SONAR/Analysis+Parameters)

\[5\] https://stackoverflow.com/questions/32047585/jenkins-sonar-github-integration

\[6\] [Docker启动SonarQube错误es137](https://github.com/10up/wp-local-docker/issues/6)

\[7\] https://yq.aliyun.com/articles/316487