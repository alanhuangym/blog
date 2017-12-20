---
title: MongoDB 复制集
date: 2017-09-12 14:24:00
tags:
- mongodb
categories:
- mongodb
---

### MongoDB复制集开启（本机）

在本地开启三个mongod，当作三个服务器，分别使用作**主复制集、从复制集、仲裁复制集**

```
127.0.0.1:28001   //主复制集
127.0.0.1:28002   //从复制集
127.0.0.1:28003   //仲裁复制集
```



**1.首先建立三个文件夹分别存放数据库数据文件。**

**2.分别建立各自的conf配置文件**

配置内容如下：（三个服务器只是端口不同，其他配置相同）

/data/mongodb/conf/28001.conf

```
# 端口
port=28001
# 绑定ip地址
bind_ip=127.0.0.1
# 日志文件路径
logpath=/data/mongodb/log/28001.log
# 数据文件存放目录
dbpath=/data/mongodb/data/28001/
# 以追加的方式写日志
logappend=true

pidfilepath=/data/mongodb/data/28001/28001.pid
fork=true
oplogSize=1024
# 复制集名称
replSet=MyMongo
```

其中复制集名称最重要，因为如果需要放在同一个复制集，必须保证不同的服务器的复制集名称完全相同。（<font color="red">其中replSet的S记得要大写</font>）

**3.启动三个服务器**

```
mongod -f /data/mongodb/conf/28001.conf
mongod -f /data/mongodb/conf/28002.conf
mongod -f /data/mongodb/conf/28003.conf
```

后面跟的是配置文件的地址，启动完成后，会出现一个fork:successful的提示信息，而且不是在前台启动的，所以要注意，可能已经启动过了。

查找是否有启动可以使用以下命令查看

```
ps -ef | grep mongo
```

**4.初始化复制集**

如果是第一次进行初始化，进入需要成为主复制集的服务器，这里进入28001端口

进入mongo客户端

```
mongo localhost:28001
```

然后输入初始化命令

```
rs.initiate({_id:'MyMongo',members:[{_id:1,host:'127.0.0.1:28001'}]})
```

第一个_id：复制集的名称

member：复制集服务器列表（因为其他服务器还没完成初始化，所以只有一个当前的服务器）

第二个_id：指定给服务器的ID

host：服务器主机地址

**5.添加从复制集合仲裁复制集**

完成初始化，可以看到，当前服务器的名称已经成为了primary。因为是整个复制集中第一个完成初始化的，所以该服务器成为主复制集。

然后我们需要添加从复制集：

```
rs.add('127.0.0.1:28002') 
```

{”ok“ : 1}添加成功

调用rs.status()可以查看到复制集的情况

然后我们添加仲裁复制集：

```
rs.addArb('127.0.0.1:28003')
```

{”ok“ : 1}添加成功

**6.测试**



Ref:

\[1\] http://www.cnblogs.com/nicolegxt/p/6841442.html?utm_source=itdadao&utm_medium=referral

\[2\] http://blog.csdn.net/lichangzai/article/details/50903130