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

### 1. docker简介

安装docker就不介绍了，可以直接从[官网](https://www.docker.com)按照指引安装

##### 1.1 docker的一些基础概念

- **镜像（image）**：相当于docker容器启动的母版，docker启动的都是从镜像延伸出来的容器，可以自己构建和存储在镜像仓库
- **容器（container）**：由单一服务构成的针对最基本功能实现的 **服务（service）**（比如Nginx），docker推荐的是一个容器一个服务

![](http://ondsf10qe.bkt.clouddn.com/jenkins18.png)

##### 1.2 docker基础操作

docker的具体操作可以查看[官方文档](https://docs.docker.com/)或者命令行`docker —help`，这里介绍一些比较常用的

**容器生命周期管理**

- `docker run -p 80:80 -v /data:/data -d nginx:latest `

  创建一个新的容器并运行一个命令，通常使用是`-p 80:80` 将宿主机的80端口与docker容器的80端口绑定，`-v /data:/data`将宿主机的文件路径与docker容器内部的文件相映射，通常用来存储容器的数据，从而达到增量更新和缓存的目的，`-d`是后台运行并返回新创建容器的id，`nginx:latest`是镜像名和标签

- `docker start` 启动一个或多少已经被停止的容器

  `docker stop` 停止一个运行中的容器

  `docker restart`重启容器

- `docker rm`移除一个容器，当容器停止之后不会消失，需要手动删除

**容器操作**

- `docker ps`查看当前正在运行的容器，`-a`可以查看所有容器包括未运行的

**本地镜像管理**

- `docker build -t 创建的镜像名称和标签 文件路径`使用文件路径的Dockerfile创建镜像，如果镜像名称和标签有重复的，则最新的镜像会使用该名称和标签，旧的镜像将变成\<none\>，在后述操作当中可以批量删除。
- `docker rmi`移除镜像

**镜像仓库**

- `docker push`
- `docker pull`

##### 1.3 dockerfile构建

docker可以直接使用官方的镜像，包含基础运行程序的环境，但如果希望创建新的镜像，可以使用dockerfile进行构建。

```dockerfile
# 设定镜像的官方运行环境
FROM google/nodejs
# 设定docker内部工作环境为根目录下的app文件夹
WORKDIR /app
# 添加宿主机中的package.json文件进入docker内app文件夹
ADD package.json /app/
# 在docker内部运行npm install命令，根据需要安装第三方库
RUN npm install
# 添加宿主机当前目录的所有文件进入docker
ADD . /app
# 暴露docker的8000端口
EXPOSE 8000
# 不运行命令
CMD []
# 启动服务
ENTRYPOINT ["/nodejs/bin/npm", "start"]
```

##### 1.4 docker compose 与jenkins类似

##### 1.5 docker 私有仓库搭建

仓库的搭建也不难，只用使用官方的registry docker即可。

启动代码`sudo docker run -d -p 5000:5000 -v /data:/tmp/registry —restart=always --name `

即在5000端口启动了一个仓库docker

<font color="red">*</font>一般启动registryd docker会遇到https安全传输的问题。

一般当尝试向服务器push镜像时，会出现错误

```
Forbidden. If this private registry supports only HTTP or HTTPS with an unknown CA certificate, please add –insecure-registry 10.0.0.26:5000 to the daemon’s arguments. 
```

解决这个问题的方法见[StackOverflow](http://stackoverflow.com/questions/26710153/remote-access-to-a-private-docker-registry)
最快最简单的方法是将如下行添加到`/etc/default/docker`，然后重启Docker Daemon：
`DOCKER_OPTS="--insecure-registry localhost:5000"`

##### 1.6 Swarm集群拓展

##### 1.7 小问题：

- 当docker启动后，希望添加docker与宿主机的端口映射

  docker容器的操作变换很方便快捷，同时消耗比较少，所以如果希望更改端口映射，不建议手工添加，最好还是重新开启一个容器，使用docker官方的用法`-p 5000:5000`

  如果容器内有工作内容需要保存，可以commit一个docker image保存数据，然后再启动一个新的容器


- docker容器启动的后的进入方法

  docker还是提倡一个容器一个进程的理念，但是有时还是需要进入docker容器中，进行一些配置，所以我们需要一个方法进入已经启动的docker容器内，同时需要一个终端进行交互

  - docker attach
  - docker 第三方工具(nsenter、nsinit)
  - docker exec<font color=“red”>（推荐）</font>

##### 1.8 Docker镜像仓库国内加速

http://www.cnblogs.com/anliven/p/6218741.html

### ~~2.APACHE服务器~~



### 3.Jenkins持续化集成

介绍完docker和apache，我们希望能够通过jenkins进行一个集成，能够自动化实现在docker容器中的apache服务器的重启操作。

![](http://ondsf10qe.bkt.clouddn.com/jenkins16.png)

##### 3.1 在jenkins中使用docker有两种方法

1. 为jenkins添加用户权限，直接在命令行执行
2. 使用docker build step plugin 插件进行docker操作

因为插件操作比较麻烦，所以我们选择给jenkins添加用户权限，从而直接在命令行中编写脚本

<font color="red">*</font>为macos添加jenkins用户权限

```
# 使用id查看自己的群组和用户名称
$ id
uid=501(alan) gid=20(staff)  ...

# 获取之后，先停止jenkins服务
$ sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
```

然后通过命令行或者直接修改`/Library/LaunchDaemons/org.jenkins-ci.plist`文件



![](http://ondsf10qe.bkt.clouddn.com/jenkins20.png)

```
# 然后添加权限
$ sudo chown -R userName /Users/Shared/Jenkins
$ sudo chown -R userName /var/log/jenkins

#重启Jenkins
$ sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
```

##### 3.2 编写jenkins设置

之前的设置跟[第一篇](http://pirrla.cn/2017/12/18/jenkins/jenkins1/)设置一样，设定好每分钟自动从代码库获取更新

然后到了构建操作脚本

![](http://ondsf10qe.bkt.clouddn.com/jenkins17.png)

```shell
#!/bin/sh
id
set +e
echo '>>> Get old container id'

# 查找正在运行的docker容器
CID=$(/usr/local/bin/docker ps | grep "my_nodejs" | awk '{print $1}')
# 获取使用“my_nodejs”镜像的容器id
echo $CID

# 根据新的代码，构建新的docker
/usr/local/bin/docker build -t my_nodejs .

# 停止旧的容器
if [ "$CID" != "" ];then
  /usr/local/bin/docker stop $CID
fi

# 启动新容器
/usr/local/bin/docker run -p 8000:8000 -d my_nodejs

# 删除旧容器
/usr/local/bin/docker rm $CID
# 删除旧镜像

```

这样就完成了一个自动化部署docker的操作。

![](http://ondsf10qe.bkt.clouddn.com/jenkins15.png)

References:

\[1\] [使用Jenkins来构建Docker容器](http://www.cnblogs.com/Leo_wl/p/4314792.html)

\[2\] [Docker 教程](http://www.runoob.com/docker/docker-tutorial.html)

\[3\] [Docker启动后的端口映射](http://blog.huangang.net/2017/01/06/docker%E5%AE%B9%E5%99%A8%E5%90%AF%E5%8A%A8%E5%90%8E%E6%B7%BB%E5%8A%A0%E7%AB%AF%E5%8F%A3%E6%98%A0%E5%B0%84/)

\[4\] [进入docker容器的方法](http://blog.csdn.net/woshiluahuo/article/details/52239407)

\[5\] [如何进入Docker容器](http://blog.csdn.net/u010397369/article/details/41045251)

\[6\] https://blog.catscarlet.com/201612022593.html

\[7\] http://ju.outofmemory.cn/entry/306621

\[8\] https://www.qcloud.com/community/article/164816001481011806

\[9\] https://www.jianshu.com/p/41f2def6ec59

\[10\] [Mac Jenkins 权限问题](http://www.cnblogs.com/ihojin/p/jenkins-permission.html)

\[11\] https://blog.catscarlet.com/201612022593.html

\[12\] [Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)

\[13\] [Mac Jenkins 修改端口](http://blog.51cto.com/tangoo/1435078)