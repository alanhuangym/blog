---
title: Elasticsearch笔记
date: 2017-07-14 09:28:34
tags:
- elasticsearch
categories:
- elasticsearch
---

### **Elasticsearch 5.5 [文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)**

### [Set up Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html)

* ES目录下config/elasticsearch.yml 是ES的配置文件

配置如下：详细设置看[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)

```yaml
# Cluster
# 设置cluster的名称
cluster.name: my-application
# Node
# 设置node的名称
node.name: node-1
# 添加node自定义属性
node.attr.rack: r1
# 存储数据的路径
# Path to log files:
path.logs: /path/to/logs
...
```

如需要在ES启动时，再输入node名称等信息可以以下配置：

```yaml
#text是直接输入数据并显示
node:
  name: ${prompt.text}
#secret则是输入的数据不会在终端上显示
node:
  name: ${prompt.secret}
```

```
Enter value for [node.name]:
```

这样在启动ES时就需要输入名称了。

* ES目录下config/log4j2.properties 是ES的日志配置文件

注意日志配置文件对多余的<font color=red>空格</font>会解析错误，所以需要保证没有多余的空格。



### [Set up X-Pack](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-xpack.html)

### [API Conventions](https://www.elastic.co/guide/en/elasticsearch/reference/current/api-conventions.html)

##### Multiple Indices

后文所用的API基本都涉及到**索引名称**

* _all 指所有的索引
* 索引名称同时支持通配符，如test*,te\*t
* 索引名称还支持筛选，如+test*,-test3

包含多个索引操作的API支持以下参数：

* ignore_unavailable - 是否忽略索引如索引名称不存在
* allow_no_indices - 当通配符匹配失败后，是否整条request都失败
* expand_wildcards - open/close 设置通配符匹配的索引的开关状态

##### Date math support in index names

如果需要根据日期去匹配索引，可以使用以下通配符：

```
<static_name{date_math_expr{date_format|time_zone}}>
```

* static_name - 固定的文字部分
* date_math_expr - 日期表达式，通常使用now/d
* date_format - 日期的显示格式
* time_zone - 时区

例子：

| Expression                              | Resolves to           |
| --------------------------------------- | --------------------- |
| `<logstash-{now/d}>`                    | `logstash-2024.03.22` |
| `<logstash-{now/M}>`                    | `logstash-2024.03.01` |
| `<logstash-{now/M{YYYY.MM}}>`           | `logstash-2024.03`    |
| `<logstash-{now/M-1M{YYYY.MM}}>`        | `logstash-2024.02`    |
| `<logstash-{now/d{YYYY.MM.dd|+12:00}}>` | `logstash-2024.03.23` |

其中，由于url需要进行标准化，所以需要对符号进行百分号化

```
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

百分号编码如下：

| `<`  | `%3C` |
| ---- | ----- |
| `>`  | `%3E` |
| `/`  | `%2F` |
| `{`  | `%7B` |
| `}`  | `%7D` |
| `|`  | `%7C` |
| `+`  | `%2B` |
| `:`  | `%3A` |
| `,`  | `%2C` |

##### Common options

* 美化输出，可以使用 `?pretty=true`参数
* 为了将时间、大小（1h,1kb）等变得可读，可以使用`?human=true`参数
* 日期时间
* - +1h = 加一小时
  - -1d = 减一天
  - /d = 取整最近的一天

具体的日期表达式如下：

| 符号       | 代表           |
| -------- | ------------ |
| `y`      | years        |
| `M`      | months       |
| `w`      | weeks        |
| `d`      | days         |
| `h`      | hours        |
| `H`      | hours        |
| `m`      | minutes      |
| `s`      | seconds      |
| `ms`     | milliseconds |
| `micros` | microseconds |
| `nanos`  | nanoseconds  |

* 错误回溯，可以使用`?error_trace=true`参数



### [Document APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)

##### Reading and Writing documents

**基础写操作**：

首先，每一个索引操作来到ES都会首先被送去不同的复制组（复制组指的是ES内含有很多的分片，包括主分片和主分片的复制分片，每一个主分片和其复制分片构成一个复制组）。而分组方法，则是通过路由（路由一般指将文档的ID进行哈希操作，从而根据结果分配到不同的复制组中）分配到不同的复制组中。

进入到一个复制组中，操作将会发送给主分片，主分片将进行以下操作：

1. 验证来到的操作，如操作非法则拒绝此操作；
2. 在主分片内进行本地操作，这同样会检查数值域的合法性，如果非法也会拒绝操作；
3. 转发这个操作至复制组中的复制分片，这个操作将是平行进行的；
4. 当整个复制组的复制分片都已经完成了操作，将会通知主分片，主分片将会通知客户端操作的完成性。

失败处理：

* 当主分片崩溃了，主持节点将会将一个复制分片提升为主分片
* 当主分片转发操作给复制分片时，将会检查自己是否还是主分片，否则将造成数据不同步
* 当主分片转发操作给复制分片，但复制分片崩溃了，主分片将会对master节点进行报告，同时会开始进行新的复制分片的制作流程

**基础读操作**

读操作相对于写操作来说，将轻量很多。

当一个节点解说道了读操作，节点将操作转发到该操作的相关分片中，对应的分片将对该操作负责，不需要一定要主分片进行操作：

1. 解析读操作，并转发给相关的复制组；
2. 从复制组选择任一活动的分片，主分片或复制分片都可以；
3. 将分片领域的操作进行执行；
4. 将结果进行汇总，并呈献给客户端。

~~待续。待继续整理~~。

##### Index API

基础操作

```json
PUT twitter/tweet/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

当该索引不存在时，系统会自动创建索引，并且自动设置mapping。如不想要系统自动创建索引，那么可以将参数`action.auto_create_index`设置为`false`。

同时，如果不希望自动设置mapping(设置各个域的数值类型和分析器等参数)，可以在索引创建时设置settings`index.mapper.dynamic`设置为`false`。

版本控制

每一次创建文档成功，文档将会自带一个`version`字段。通过该字段的控制，则可以实现版本控制。

```json
PUT twitter/tweet/1?version=2
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

当需要系统自动创建文档ID时可以使用**POST**方法：

```json
POST twitter/tweet/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```



======================================================

以下为简单笔记

GET 获取信息，HEAD检测是否存在。

批量搜索删除

```json
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
```

更新数据

```json
POST test/type1/1/_update
{
    "script" : {
        "inline": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

批量筛选更新：

```json
POST twitter/_update_by_query
{
  "script": {
    "inline": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

reindex

```json
POST _reindex
{
  "source": {
    "index": "twitter",
    "type": "tweet",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

完整的source_filter 支持通配符

```json
GET /_search
{
    "_source": {
        "includes": [ "obj1.*", "obj2.*" ],
        "excludes": [ "*.description" ]
    },
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

自定义字段，并输出

```json
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "script_fields" : {
        "test1" : {
            "script" : {
                "lang": "painless",
                "inline": "doc['my_field_name'].value * 2"
            }
        },
        "test2" : {
            "script" : {
                "lang": "painless",
                "inline": "doc['my_field_name'].value * factor",
                "params" : {
                    "factor"  : 2.0
                }
            }
        }
    }
}
```

[选择在什么分片执行操作](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-preference.html)

```json
GET /_search?preference=xyzabc123
{
    "query": {
        "match": {
            "title": "elasticsearch"
        }
    }
}
```

agg-[cardinality](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-cardinality-aggregation.html)

对多set的可能会出错，对少的比较正确

聚合例子

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "t_shirts" : {
            "filter" : { "term": { "type": "t-shirt" } },
            "aggs" : {
                "avg_price" : { "avg" : { "field" : "price" } }
            }
        }
    }
}
```

自动建立条状图聚类

```json
POST /sales/_search?size=0
{
    "aggs" : {
        "prices" : {
            "histogram" : {
                "field" : "price",
                "interval" : 50,
                "min_doc_count" : 1
            }
        }
    }
}
```

聚类 stats字段一键显示各种avg/sum/…



In order to disable allowing to delete indices via wildcards or `_all`, set `action.destructive_requires_name` setting in the config to `true`. 



[转移不同index内的数据（index太大）](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-rollover-index.html#indices-rollover-index)



添加alias

```json
POST /_aliases
{
    "actions" : [
        { "add" : { "indices" : ["test1", "test2"], "alias" : "alias1" } }
    ]
}
```

移除alias

```json
POST /_aliases
{
    "actions" : [
        { "remove" : { "index" : "test1", "alias" : "alias1" } }
    ]
}
```

索引模板

```json
PUT _template/template_1
{
  "template": "te*",
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "type1": {
      "_source": {
        "enabled": false
      },
      "properties": {
        "host_name": {
          "type": "keyword"
        },
        "created_at": {
          "type": "date",
          "format": "EEE MMM dd HH:mm:ss Z YYYY"
        }
      }
    }
  }
}
```

三倍重要搜索

```json
GET /_search
{
  "query": {
    "multi_match" : {
      "query" : "this is a test",
      "fields" : [ "subject^3", "message" ] 
    }
  }
}
```

多个field同用一个名字

```json
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "first_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "last_name": {
          "type": "text",
          "copy_to": "full_name" 
        },
        "full_name": {
          "type": "text"
        }
      }
    }
  }
}
```

master node 个数

最少discovery.zen.minimum_master_nodes=(master_eligible_nodes / 2) + 1

脚本语言（默认painless）

| Language                                 | Sandboxed                                | Required plugin                          |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [`painless`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html) | yes                                      | built-in                                 |
| [`groovy`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-groovy.html) | [no](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-security.html) | built-in                                 |
| [`javascript`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/lang-javascript.html) | [no](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-security.html) | [`lang-javascript`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/lang-javascript.html) |
| [`python`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/lang-python.html) | [no](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-security.html) | [`lang-python`](https://www.elastic.co/guide/en/elasticsearch/plugins/5.5/lang-python.html) |



By far the fastest most efficient way to access a field value from a script is to use the `doc['field_name']` syntax,



基本概念**

一个cluster是一个或多个node的集合，一个node可以存储index，一个index可以有多个type。

**索引**

不输入id

```json
POST /customer/external
{
  "name": "Jane Doe"
}
```

自定id

```json
PUT /customer/external/2
{
  "name": "Jane Doe"
}
```

**更新数据**

```json
POST /customer/external/1/_update
{
  "script" : "ctx._source.age += 5"
}
```

Note that as of this writing, updates can only be performed on a single document at a time. In the future, Elasticsearch might provide the ability to update multiple documents given a query condition (like an `SQL UPDATE-WHERE` statement).

**批量操作**

```json
POST /customer/external/_bulk
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

[批量操作日期格式的index](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-math-index-names.html)