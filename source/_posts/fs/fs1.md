---
title: 文件系统1
date: 2017-09-05 14:23:06
tags:
- filesystem
categories:
- filesystem
---

### 数据库选择

**关系型数据库 - Oracle**

基础事务有四个特性：

- 原子性：整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回复 （Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- 一致性：在事务开始之前和事务结束以后，数据库的完整性限制没有被破坏。
- 隔离性：两个事务的执行是互不干扰的，一个事务不可能看到其他事务运行时，中间某一时刻的数据。
- 持久性：在事务完成以后，该事务对数据库所作的更改便持久地保存在数据库之中，并不会被回复。



对网站来说，关系型数据库的很多特性不再需要了：

- 事务一致性

关系型数据库在对事物一致性的维护中有很大的开销，而现在很多web系统对事物的读写一致性都不高

- 读写实时性

对关系数据库来说，插入一条数据之后立刻查询，是肯定可以读出这条数据的，但是对于很多web应用来说，并不要求这么高的实时性，比如发一条消息之后，过几秒乃至十几秒之后才看到这条动态是完全可以接受的

- 复杂SQL，特别是多表关联查询

任何大数据量的web系统，都非常忌讳多个大表的关联查询，以及复杂的数据分析类型的复杂SQL报表查询，特别是SNS类型的网站，从需求以及产品阶级角度，就避免了这种情况的产生。往往更多的只是单表的主键查询，以及单表的简单条件分页查询，SQL的功能极大的弱化了

**NoSQL - MongoDB**

由于在本项目中，数据是新闻资讯的存储，而新闻资讯相互之间是相对独立的，所以不需要使用到关系型数据库。

非关系型数据库相当于是简化版的关系型数据库，减少了一些相对少用到的功能，提升了产品的性能。相对于关系型数据库，抛弃了ACID特性。

但是由于是基于键值对的特性，所以非常容易进行扩展，可拓展性强。



### 文件系统

但是MongoDB也有一个问题。由于MongoDB的文档结构为BJSON格式（BJSON全称：Binary JSON），而BJSON格式本身就支持保存二进制格式的数据，因此可以把文件的二进制格式的数据直接保存到MongoDB的文档结构中。但是由于一个BJSON的最大长度不能超过16M，所以限制了单个文档中能存入的最大文件不能超过<font color = "red">16M</font>

为了提供对大容量文件存取的支持，samus驱动提供了“GridFS”方式来支持。

**BJSON**

[python 实现](http://blog.csdn.net/kwsy2008/article/details/48969607)

**GridDFS**

MongoDB内置功能。

GridFS是一种在MongoDB中存储大二进制文件的机制。使用GridFS存文件有如下几个原因：

利用Grid可以简化需求。要是已经用了MongoDB，GridFS就可以不需要使用独立文件存储架构。

基本原理是将文件存储在两个Collection之中，一个保存文件索引，一个保存文件内容。4MB每一块。



| 指标        | 适合类型      | 文件分布             | 系统性能       | 复杂度  | FUSE | POSIX | 备份机制           | 通讯协议接口   | 社区支持  |      | 开发语言 |
| --------- | --------- | ---------------- | ---------- | ---- | ---- | ----- | -------------- | -------- | ----- | ---- | ---- |
| FastDFS   | 4KB~500MB | 小文件合并存储不分片处理     | 很高         | 简单   | 不支持  | 不支持   | 组内冗余备份         | ApiHTTP  | 国内用户群 |      | C语言  |
| TFS       | 所有文件      | 小文件合并，以block组织分片 |            | 复杂   | 不支持  | 不支持   | Block存储多份,主辅灾备 | APIhttp  | 少     |      | C++  |
| MFS       | 大于64K     | 分片存储             | Master占内存多 |      | 支持   | 支持    | 多点备份动态冗余       | 使用fuse挂在 | 较多    |      | Perl |
| HDFS      | 大文件       | 大文件分片分块存储        |            | 简单   | 支持   | 支持    | 多副本            | 原生api    | 较多    |      | java |
| Ceph      | 对象文件块     | OSD一主多从          |            | 复杂   | 支持   | 支持    | 多副本            | 原生api    | 较少    |      | C++  |
| MogileFS  | 海量小图片     |                  | 高          | 复杂   | 可以支持 | 不支持   | 动态冗余           | 原生api    | 文档少   |      | Perl |
| ClusterFS | 大文件       |                  |            | 简单   | 支持   | 支持    | 镜像             |          | 多     |      | C    |

**HDFS**

还是适合大数据处理，附件等文档太小，不适合使用HDFS。

**FastDFS**

1. 为互联网量身定制，海量数据文件存储。
2. 高可用(同组备份机制)。
3. FastDFS不是通用的文件系统，只能通过api来访问，目前提供c,java,php客户端。phtyon由第三方开发者提供。
4. FastDFS可以看作是基于key/value pair存储系统，也许称为分布式文件存储服务更合适。
5. 支持高并发(这个好像没体现出支持什么高并发,这个是nginx的功劳吧)



### 从MongoDB到ElasticSearch

MongoDB 本身是自带文本索引功能的，但是，不支持中文。**术业有专攻**，MongoDB 是数据存储应用，那么全文检索就使用专业的全文搜索引擎吧。

**0.River**

由于只支持ES 2.0 以下版本，已弃用。

**1.[Mongo-Connector](https://github.com/mongodb-labs/mongo-connector)**

实时

- MongoDB必须开启复制集[2]

This assumes there is a MongoDB replica set running on port 27017 and that Elasticsearch is running on port 9200 both on the local machine.

- 安装mongo-connector

```
pip install 'mongo-connector[elastic5]'
```

- 运行mongo-connector

```
mongo-connector -m 127.0.0.1:27017 -t 127.0.0.1:9200 -d elastic_doc_manager
```

启动之后，现在开始，在mongoDB上的操作，都会同步到elasticsearch之中了。

（对旧数据无影响）

问题：根据其他用户的评价，mongo-conncetor会出现延迟和退出的现象

**2.[mongodb to elasticsearch](https://github.com/keenwon/mongodb-to-elasticsearch)**

非实时，应对一天更新一次的场景

使用了mongoose

一个node.js写的小程序，可以将mongodb的数据输入到es。



Ref:

\[1\] [MongoDB 数据自动同步到 ElasticSearch](https://segmentfault.com/a/1190000003773614)

\[2\] [副本集设置](http://www.runoob.com/mongodb/mongodb-replication.html)