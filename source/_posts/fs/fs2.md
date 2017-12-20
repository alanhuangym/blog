---
title: 文件系统2
date: 2017-09-12 14:23:06
tags:
- filesystem
categories:
- filesystem
---

### FastDFS与GridFS的选择

#### FastDFS

FastDFS服务器端有两个角色：追踪器（tracker）和存储节点（storage）。

追踪器：主要负责调度工作，在访问上起到**负载均衡**的作用。

存储节点：存储文件，完成文件管理的存储、同步和提供存取接口等功能。

FastDFS同时会对文件的metadata（相关属性，例如宽度等）进行管理，以键值对的方式表示。

建议文件大小： 4KB < file_size <500MB

系统结构：

![](http://static.oschina.net/uploads/img/201204/20230218_pNXn.jpg)

存储节点采用分卷的组织方式。存储系统由一个或多个卷组成，卷与卷之间的文件是相对独立的。

一个卷可以由一台或多台存储服务器组成，多台存储服务器可以起到冗余备份和负载均衡的作用。

**上传文件交互过程：**

1. client询问tracker上传到的storage，不需要附加参数；
2. tracker返回一台可用的storage；
3. client直接和storage通讯完成文件上传。 

**下载文件交互过程：**

1. client询问tracker下载文件的storage，参数为文件标识（卷名和文件名）；
2. tracker返回一台可用的storage；
3. client直接和storage通讯完成文件下载。



#### GridFS

MongoDB单个document存储上限为16M，为了存储大于16M的文件，我们可以使用MongoDB官方的文件系统GridFS。

默认分割大小**256K**。

GridFS的思想很简单就是将一个很大的文件进行分割，分割后的每一个小文件作为一个document单独存储。然后提供了两个集合来存储分割的信息和文件的元信息。这两个集合默认是fs.files和fs.chunks。

fs.files集合就存储了这个文件的基本信息：

- \_id
- length: 文件的大小
- filename: 文件的名称
- chunkSize: 这个文件每一个分块的大小，单位是字节，默认为256K。
- uploadDate: 文件上传的时间。
- md5: MD5信息，以检查文件完整性

fs.chunks集合包括以下几个字段：

- \_id
- n: 表示是分块中的第几块，从0开始。
- data: 存储的是当前文件分块的二进制数据。
- file_id: 表示的是该块是属于哪一个文件，存储的就是fs.files中document的\_id。在读取文件的时候，先在fs.files集合中找满足条件的document，获取它的_id值，然后根据这个值到fs.chunks集合中查找所有files_id为该值的document，并按n排序，最后依次读取document中data的值拼凑出原来的文件。

### Mongo Connector使用心得

当按照[复制集开启流程](http://pirrla.cn/2017/09/05/fs/fs1/)完成后，即可使用mongo connector。

1. 开启MongoDB复制集
2. 开启ES
3. 开启mongo connector

```
sudo mongo-connector --auto-commit-interval=0 -m localhost:28001 -t localhost:9200 -d elastic2_doc_manager -n news.tj_reports -o /Users/alan/Downloads/oplog.txt
```



<font color="red">之前的表述有错误，开启mongo connector后数据会直接同步，旧数据也会</font>

**MongoDB的database会作为es的index**

**collection会作为es的type**

然后各个字段都会同步到es中。（增删改都会，version会变）





### Mongo connector support for GridFS[2]

mongo connector支持将MongoDB的GridFS中的文件复制到es中，通过es的mapping种类attachment。(~~已过期~~)

1. ES安装[attachment插件](https://github.com/elastic/elasticsearch-mapper-attachments)


2. ​

attachment type 已经被[抛弃](https://www.elastic.co/guide/en/elasticsearch/plugins/5.6/mapper-attachments.html#mapper-attachments-install)？

<font color="red">5.6ES推出了一个[Ingest Attachment Processor](https://www.elastic.co/guide/en/elasticsearch/plugins/5.6/ingest-attachment.html)</font>





Ref:

\[1\] http://www.oschina.net/p/fastdfs

\[2\] https://github.com/mongodb-labs/mongo-connector/wiki/Usage%20with%20ElasticSearch

\[3\] [FastDFS 介绍](http://blog.csdn.net/WK313753744/article/details/49943155)

