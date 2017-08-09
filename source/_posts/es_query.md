---
title: ES搜索笔记
date: 2017-08-04 10:08:20
tags:
- es
categories:
- es
---

首先，了解[QUERY结构体]([Request Body Search](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html))的写法：

### 1.最基础的是**"Query"字段**

它包含最基础的搜索内容和其他设置。(后面再具体介绍内容写法)

```json
//最简单的结构体
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

##### 2.**from 和 size 字段**

控制显示搜索结果的数量和页数

##### 3.**sort 字段**

例子如下

```json
GET /my_index/my_type/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

通过一个列表，控制排列顺序，如果遇到相同的则比较下一项。

##### 4.**\_source **

由于es的索引不是所有字段我们都需要，所以需要进行字段筛选，同时由于使用了highlight功能，所以我们可能只需要将hightlight字段筛选出来即可。（可使用正则表达式匹配字段名）

##### 5.**script_fields**

用于编写函数，并返回到搜索结果中

##### 6.**highlight**

用于高亮显示某些搜索结果，例子如下

```json
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "_all" : {}
        }
    }
}
```

##### 7.rescore

用于在返回的搜索结果中再次进行得分计算，并重新计算得分，可用于在搜索相关性后，在根据资讯的日期进行二次排序

##### 8.search type

用于决定是全局搜索并计算得分，还是在各自的分片上进行分布式得分计算和搜索

##### 9.scroll

设定搜索结果的存活时间，避免重复搜索浪费资源，只用于在可以滑动搜索窗口的客户端中，如Python

##### 10.preference

分片喜好

##### 11.version

返回每一个hit的版本号

##### 12.min_score

只有超过某个数值的_score才会返回在搜索结果中

##### 13.collapse

selecting only the top sorted document per collapse key

用于收集某一个关键词的最高分的文档，例如检索每个用户最多like的tweet（必须包含sort）

```json
GET /twitter/tweet/_search
{
    "query": {
        "match": {
            "message": "elasticsearch"
        }
    },
    "collapse" : {
        "field" : "user" 
    },
    "sort": ["likes"], 
    "from": 10 
}
```

##### 14.search\_after

也用于翻页，将本页的最后一个搜索结果填入参数则进入下一页（from必须是0）



| 字段            | 是否需要                                     |
| ------------- | ---------------------------------------- |
| query         | 必须，基础字段（后表续query字段具体写法）                  |
| from/size     | 需要，用于管理用户的搜索页的页码                         |
| sort          | 不需要，用于控制排列顺序，首先是比对\_score字段，如果相同则比较date，日期较新的排列较前（但是由于我们是match搜索，非term搜索，所以出现\_score相同的情况较少，不能使用此字段） |
| _source       | 需要，进行字段筛选                                |
| script_fields | 不需要，暂时不需要编写函数                            |
| highlight     | 需要，进行搜索字段标红                              |
| rescore       | 可能需要                                     |
| search_type   | 需要                                       |
| scroll        | 可能需要                                     |
| preference    | 不需要                                      |
| version       | 可能需要                                     |
| min_score     | 不需要                                      |
| collapse      | 可能需要                                     |
| search_after  | 可能需要                                     |



### Query字段内部的编写方式

叶语句：

几乎语句都能写成match的形式，所以使用match

| match方法             | 是否采用 |
| ------------------- | ---- |
| match               |      |
| match_phrase        |      |
| multi_match         |      |
| query_string        |      |
| simple_query_string |      |

聚合语句：

bool

匹配到越多，越高分，不同的match之间得分相加

dis_max

是disjunctionmax 各自的match执行，然后最高分的为最终得分，通过设置tie_breaker来调参

function score

参考[前文](http://pirrla.cn/2017/07/12/es_similarity/)

我认为的搜索语句：

```json
GET /windxw/_search?search_type=dfs_query_then_fetch
{
  "from": 0,
  "size": 20,
  "_source": ["title","content"], 
  "highlight": {
    "pre_tags": ["<font color=\"red\">"],
    "post_tags": ["</font>"],
    "fields": {
      "content": {},
      "title": {}
    }
  }, 
  "query": {
    "bool": {
      "should": [
        {"match": {
          "title": {
            "query": "uber",
            "boost":2，
            "operator": "and"
          }
        }},
        {"match": {
          "content": {
            "query": "uber",
            "boost":2，
            "operator": "and"
          }
        }},
        {"match": {
          "keywords": {
            "query": "uber",
            "boost":1，
            "operator": "and"
          }
        }}
      ]，
      "minimum_should_match": 2 
    }
  }
}
```

