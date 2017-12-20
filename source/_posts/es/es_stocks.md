---
title: Elasticsearch股票分析
date: 2017-07-13
tags:
- elasticsearch
categories:
- elasticsearch
---

### 1.制作股票代号-股票名称同义词词典，并添加入路径（同时需要将股票名称添加进拓展词典）

首先，通过Python，我们将收集股票名称和股票代码来制作同义词词典。

股票数据来源来自于[东方财富网](http://quote.eastmoney.com/stocklist.html)。

```python
import requests
from bs4 import BeautifulSoup
#用两个列表分别存储上证和深证各支股票的信息
sh = []
sz = []
with requests.Session() as s:
    html = s.get(r'http://quote.eastmoney.com/stocklist.html')
    soup = BeautifulSoup(html.content, 'lxml')
    sh_stocks =soup.find_all('ul')[7]
    for i in sh_stocks.find_all('li'):
        sh.append(i.a.text)
    sz_stocks =soup.find_all('ul')[8]
    for i in sz_stocks.find_all('li'):
        sz.append(i.a.text)
```

输出的本文结果为如下格式

```
...
501001,财通精选
501002,能源互联
501003,上海改革
501005,精准医疗
...
510190,龙头etf
510210,综指etf
510220,中小etf
510230,金融etf
...
```

![](http://mednoter.com/media/files/2014/Sep/2014-09-07-flow.png)

首先，由于分词ik不能识别股票名称（不能正确分词），即使添加了同义词词典也无法检索到，所以需要在拓展词库中添加全部股票名称。

其中，由于用户输入一般是忽略大小写的，为了方便用户能够输入大小写都能匹配到股票信息，我们将拓展词典、同义词词典的股票名称信息都更改为小写，这样无论用户输入大写还是小写都能匹配到股票。

~~暂时没有找到关闭 filter * 号的方法，所以搜索不了 *stXX股票~~

~~有没有可能去除st等标记，因标记是在原股票名称前添加，不改变原股票名称~~

### 2.学习使用html_strip filter

html strip filter 代码如下：

```json
"char_filter": {
        "my_html": {
          "type": "html_strip",
          "escaped_tags": []
        }
      }
```

其中**escaped_tags**字段表示该列表之中的标签不用去除

### 3.创建索引，并设置IK为Tokenizer，Filter包括同义词和html_strip

完整设置和mapping如下：

```json
{
    "settings": {
        "index" : {
            "analysis" : {
                "analyzer" : {
                    "my_analyzer" : {
                        "tokenizer" : "my_ik",
                        "filter" : ["my_stop","synonym"],
                        "char_filter":  ["my_html"]
                    }
                },
                "char_filter": {
                    "my_html": {
                      "type": "html_strip",
                      "escaped_tags": []
                    }
                  },
                "filter" : {
                    "synonym" : {
                        "type" : "synonym",
                        "synonyms_path" : "synonym.txt"
                    },
                    "my_stop":{
                      "type":"stop",
                      "stopwords":"_none_"
                    }
                  },
                  "tokenizer":{
                  "my_ik":{
                    "type":"ik_max_word",
                    "enable_lowercase":true
                }
                }
            }
        }
    },
    "mappings":{
    "news":{ //类型名，这里为news
    "properties": {
        "title": {
            "type":      "text",
            "analyzer":  "my_analyzer"
        },
        "keywords":{
            "type":      "text",
            "analyzer":  "my_analyzer"
        },
        "content":{
            "type":      "text",
            "analyzer":  "my_analyzer"
        }
    }
    }
  }
}
```

还需要完善：

* 对于不需要和不希望加入搜索的字段进行not_analyze设置
* 设置日期字段的数值类型date

### 4.对已有数据进行reindex(约34w数据)

之前进行reindex速度明显比这次高，原因是上次没有使用analyzer。

此次reindex **34w**数据共用时**793**秒，每1秒操作**428**个文档。

如无analyzer，则每一秒操作**1581**个文档。



### 5.使用function_score进行filter检索，发布时间越靠前的得分越高

```json
GET test/news/_search?search_type=dfs_query_then_fetch
{
  "_source": ["title","publishdate"], 
  "query": {
    "function_score": {
        "query":{
          "match": {
            "title": "600340"
          }
        } ,
         "functions": [ 
            {
                    "exp": {
                        "publishdate" : {
                            "origin": "now",
                            "offset": "1h",
                            "scale" : "30d",
                            "decay": 0.2
                        }
                    },
                    "weight": 200
                }
        ],
        "boost_mode": "sum"
      }
    }
  }
```

其中使用的是[function score query](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-function-score-query.html#_supported_fields_for_decay_functions) 的 [decayfunction](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-function-score-query.html#function-decay)(衰减函数)。

各参数设置如图：

![](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/images/decay_2d.png)