---
title: Jenkins学习笔记7-Docker WebGUI管理
date: 2017/12/24
tags: 
- jenkins
categories:
- jenkins
---

### 1. Docker WebGUI管理

1. [shipyard](https://github.com/shipyard/shipyard)国产，但是20天前刚刚宣布停止更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins22.png)

2. [rancher](http://rancher.com/) 硬性条件：需要一台linux服务器做主机，至少2G内存和20G硬盘，较少更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins23.png)

3. [kevana](https://github.com/kevana/ui-for-docker)已停止更新

   ![](http://ondsf10qe.bkt.clouddn.com/jenkins24.png)

4. [portainer](https://portainer.io/)比较轻量，github显示更新及时

此处选择了portainer。

![](http://ondsf10qe.bkt.clouddn.com/jenkins19.png)



### 2.Portainer 

#### 安装

根据[官方教程](https://www.portainer.io/install.html)，使用portainer也是非常方便，使用它的官方docker镜像即可。(注意使用的是本机的9000端口，如有需要请修改)

```shell
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
`
```

<font color="red">注意：</font>`-v /var/run/docker.sock:/var/run/docker.sock`命令只能适用于linux环境

当启动完成后，portainer即在本机9000端口服务，首次进入后我们需要进行管理人员的账户创建。

![](http://ondsf10qe.bkt.clouddn.com/jenkins46.png)

之后我们就需要选择进行管理的docker环境（本地/远程）

如果是远程的话，我们需要输入目标docker服务器的地址、端口等信息，同时需要确保目标端的docker开启了TCP端口暴露设置，详情参考docker官方文档。

![](http://ondsf10qe.bkt.clouddn.com/jenkins47.png)

如果是本地的话就简单很多，直接选择连接即可，但是要注意，由于本操作需要开启portainer的docker镜像时，使用`-v /var/run/docker.sock:/var/run/docker.sock`命令才可以，但是Windows系统下无法执行该命令，所以Windows系统无法连接本地的docker。

![](http://ondsf10qe.bkt.clouddn.com/jenkins48.png)