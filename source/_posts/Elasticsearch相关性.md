---
title: Elasticsearch相关性
date: 2017-07-12
tags:
- elasticsearch
categories:
- elasticsearch
---

# ElasticSearch 相关性

### 1.简介

Elasticsearch版本：5.4.2

Kibana版本：5.4.2

Logstash版本：5.4.2

Elasticsearch在执行搜索后，返回结果的排序是根据字段**_score**决定的。排序结果是倒序排列，得分越高，越在前面。

**_score**字段即量化表明查询语句与文档内容的相似性与匹配度，为了更好地显示搜索结果，我们需要深入了解该数值的来源与相关算法，本文讨论的相关性则为此。



### 2.语句

本文通过调用**_search**的**explain**参数进行调试。

```json
GET /_search?explain
{
  "query":{
    "match":{
      "title":"a"
  }
 }
}
```



### 3.经典TF/IDF算法

在Elasticsearch 5.X 版本前，Elasticsearch默认使用的是经典的TF/IDF算法，即：

**检索词频率（TermFrequency）**

检索词在该字段出现的频率？出现频率越高，相关性也越高。 字段中出现过 5 次要比只出现过 1 次的相关性高。当然，词频不是在score计算中直接使用的。通常是取词频（TermFrequency）的平方根。

```
TF_Score = sqrt(termFreq)
```



**反向文档频率（InverseDocumentFrequency）**

每个检索词在索引中出现的频率？频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。例如中文中，“的”、“得”，“不”等词因在大多数文档中出现，所以相关性较低。

```
IDF_Score = log(maxDocs/docFreq+1)+1
```



**字段长度准则（FieldNorms）**

字段的长度是多少？长度越长，相关性越低。 检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

```
FieldNorms = 1 / sqrt(length)
```



所以最终得分为

```
Score = TF_Score*IDF_Score*FieldNorms
      = log(maxDocs / (docFreq + 1)) * sqrt(termFreq) * (1/sqrt(length))
```

*maxDocs有可能会数到已删除文档！



### 4.BM25算法

BM25是Elasticsearch5.X的默认相关性算法，它根据TF/IDF算法改进而来。它由两部分组成，分别是IDF和TFNorm.

其中，根据不同的检索方法，数值各不相同，由于Elasticsearch是分布式系统，所以默认的检索方法为Query_Then_Search（该方法为在各自分片内进行检索，然后各分片返回各自的搜索结果至协调分片，非全局检索，当数据量充足时，效果良好，但当数据量较少时，应使用全局检索，请参看后文DFS_Query_Then_Search方法）

**单词检索**（即检索内容只包含单个英语单词（空格间隔）或单个中文分词（由Tokenizer划分））：

IDF

```
log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5))
```

TFNorm

```
termfreq * (k1 + 1)) / (termfreq + k1 * (1 - b + b * fieldLength / avgFieldLength)
```

- docCount - 该索引该分片的文档总数
- docFreq - 该索引内该分片内，含有检索内容的文档的数量
- termFreq - 包含该检索内容的文档的检索内容出现次数
- k1 & b - BM25参数，常分别设为 1.2 和 0.75
- fieldLength - 包含该检索内容的文档的长度
- avgFieldLength - 该索引该分片的平均文档长度

**多词检索**

如果检索内容为一段话或多个词语，首先通过Analyzer将一段话或多个词语进行筛选和划分，包括大小写转换、停用词去除、词根筛选、同义词转换、分词（英文根据空格划分，中文根据Tokenizer划分，本文使用中文分词工具ik）。然后将得到的多个词语逐个进行单词检索，然后最终得分为多个词语的得分的总和。



### 5.其他算法

如果需要更换算法，可以在创建索引的时候，添加settings设置，更改默认的相关性算法。

```json
PUT /test
{
  "settings": {
    "index": {
      "similarity": {
        "default": {
          "type": "classic"
        }
      }
    }
  }
}
```

可选算法有

- "BM25" - 如无设置算法，则默认为BM25算法
- "classic" - 经典TF/IDF算法

除了以上的两种算法，其他的算法需要[配置参数](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-similarity.html#lm_dirichlet)

- "[DFR](http://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/similarities/DFRSimilarity.html)"  / "[DFI](http://trec.nist.gov/pubs/trec21/papers/irra.web.nb.pdf)"  / "[IB](http://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/similarities/IBSimilarity.html)" / "[LMDirichlet](http://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/similarities/LMDirichletSimilarity.html)" / "[LMJelinekMercer](http://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/similarities/LMJelinekMercerSimilarity.html)"

```json
"similarity" : {
  "my_similarity" : {
    "type" : "DFR",
    "basic_model" : "g",
    "after_effect" : "l",
    "normalization" : "h2",
    "normalization.h2.c" : "3.0"
  }
}
```

再在索引中设置mapping以应用算法

```json
{
  "book" : {
    "properties" : {
      "title" : { "type" : "text", "similarity" : "my_similarity" }
    }
}
```



### 6.Function Score

首先，function score的语句如下：

```json
GET test6/test/_search?search_type=dfs_query_then_fetch
{
  "query": {
    "function_score": {
        "query":{ //首先是检索体
          "match_all": {
          }
        },
         "functions": [  //Functions可以添加多种脚本和过滤器
          {              //使用一个数组将各个function结合起来
            "filter": {  // 过滤器
              "match": { "title": "a" } },
              "weight": 23 // weight 直接给符合过滤器的文档权重分
          },
          {
            "script_score": { //脚本打分器
              "script": {     //只能对字段类型为数值的进行数学操作
               "inline": "Math.log(2 + doc['votes'].value)"
          } 
         }
        }
        ],
      "boost":5, //将该数值影响得分
      "score_mode":"multiply", //functions内多个脚本的得分处理办法
      "boost_mode":"multiply", //检索得分和函数得分的处理办法
      "min_score":40, //返回搜索结果所需要的最少得分
      "random_score":{}, //生成一个随机数，影响得分
      "max_boost":10 //无论函数、脚本如何影响得分，最终得分最大值为此
      }
    }
  }
```

其中**boost_mode**（检索得分与函数得分d的处理方式）的模式有：

* multiply - 检索得分和函数得分的积（默认）

* sum - 检索得分和函数得分的和

* min - 检索得分和函数得分之中的较小值

* max - 检索得分和函数得分之中的较大值

* replace - 函数得分代替检索得分

  ​

而后在函数内部中不同脚本、函数和过滤器也需要进行得分处理，**score_mode**:

* multiply - 函数结果求积（默认）

* sum - 函数结果求和

* min - 函数结果最小值

* max - 函数结果最大值

* avg - 函数结果平均值

* first - 首个函数结果作为最终结果

  ​

**脚本评分**（script_score）例子中使用的是简洁版本，其完整版本为：

```json
"script_score": {
          "params": {  // 参数字段，以","分隔
            "threshold": 80,
            "discount": 0.1,
            "target": 10
          },
  		  //脚本字段，以";"分隔语句
          "script": "price  = doc['price'].value; margin = doc['margin'].value;
          if (price < threshold) { return price * margin / target };
          return price * (1 - discount) * margin / target;" 
        }
      }
```



### 7.同义词

使用同义词进行检索时，部分数据会发生变化，以BM25算法为例：

b,c为同义词

- docCount - 文档总数不变
- docFreq - 所有包含b或c的文档的数量
- termFreq - 如有一次b或c出现，则算双倍次数
- k1 & b - 不变
- fieldLength - 不变
- avgFieldLength - 每个b或c算双倍大小

### 8.DFS Query Then Fetch

可以全局搜索，当数据量较少时可以使用

```json
GET /_search?search_type=dfs_query_then_fetch
```



### 资料参考：

[1\] [Elasticsearch文档-什么是相关性？](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html)

[2\] [BM25 The Next Generation of Lucene Relevance](http://opensourceconnections.com/blog/2015/10/16/bm25-the-next-generation-of-lucene-relevation/)

[3\] [Elasticsearch文档-Similarity module](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-similarity.html)

[4\] [Elasticsearch文档-Function Score Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)

[5\] [Elasticsearch文档-DFS-DFS Query Then Fetch](https://www.elastic.co/blog/understanding-query-then-fetch-vs-dfs-query-then-fetch)