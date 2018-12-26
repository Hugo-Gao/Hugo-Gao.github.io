---
title: ElasticSearch学习一-小试牛刀
date: 2018-06-22 14:55:20
tags: [大数据,ElasticSearch]
---

> 本文根据ElasticSearch官方文档整理所得，简单介绍ElasticSearch的概念以及一些简单的检索方法

首先下载ElasticSearch和Kibana到本地，注意这两个东西的版本号要一致，然后启动Kibana，打开http://localhost:5601/app/kibana即可进入GUI的管理界面，进入Dev Tools即可操作ES。

## 1. 索引

先看ES中，索引的两个概念：

> 存储数据到 Elasticsearch 的行为叫做 *索引*(动词) ，但在索引一个文档之前，需要确定将文档存储在哪里。

> 一个 Elasticsearch 集群可以包含多个*索引* (名词)*，相应的每个索引可以包含多个*类型。*这些不同的类型存储着多个*文档* 每个文档又有多个*属性* 。

可以看到上述语句不是很通顺，主要是由于这里的索引有两个意思:

1. 名词，这里的索引就像一个数据库一样。
2. 动词，索引（动词）一个文档就是存储一个文档到一个*索引*（名词）中以便它可以被检索和查询到。就像SQL中的Insert一样。

例子：

```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

路径`/megacorp/employee/1`包含了三部分的信息：megacorp就是索引名称（像数据库）,employee(像一张表),1(特定employee，就像标识每一行的主键)。接下来的JSON就是储存的内容了。

## 2.检索文档

检索文档就像HTTP请求一样，执行一个GET请求即可取回JSON文档：

`GET /megacorp/employee/1`

类似的GET还可以改为HEAD、DELETE等。

### 2.1轻量搜索

搜索所有雇员：

`GET /megacorp/employee/_search`

查询JSON中的任意指定字段：

`GET /megacorp/employee/_search?q=last_name:Smith`

使用查询表达式搜索，它支持构建更加复杂和健壮的查询，使用请求体。：

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

### 2.2复杂的查询：

```json
GET /megacorp/employee/_search
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```

### 2.3 全文搜索

全文搜索可以完成传统关系型数据库很难做到的事情，如搜索所有喜欢攀岩的雇员：

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

这似乎与之前没什么不同啊？但是请看返回结果：

```json
"hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.53484553,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "2",
        "_score": 0.26742277,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
```

Elasticsearch 默认按照相关性得分排序，即每个文档跟查询的匹配程度。第一个最高得分的结果很明显：John Smith 的 `about` 属性清楚地写着 “rock climbing” 。但是Jane也返回了，但是由于他的关键字里并没有完全匹配，所以他的相关性没有John的高，排在后面。而传统的关系型数据库则是要么匹配要么完全不匹配。

### 2.3 短语匹配

那如果我想像关系型数据库一样只匹配完全符合的怎么办呢？就是用短语匹配的查询：`match_phrase`

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```

则只会返回完全匹配的John的文档。

### 2.4 分析查询

在使用聚合agg之前需要先对查询的字段加上fileddata=true,因为文档中写道

> Fielddata can consume a **lot** of heap space, especially when loading high cardinality `text` fields. 

fileddata会消耗大量的堆空间，所以默认是关闭的，在这里要手动打开。

```json
PUT megacorp/_mapping/employee/
{
  "properties": {
    "interests": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```



有点像Group By：如找出Employee中最受欢迎的兴趣爱好：

```json
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
},
```

在输出中可以看到,每个interest按照人数排序：

```json
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

这个aggs字段还可以作为query字段的子查询。如：

```json
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests"
      }
    }
  }
}
```

还支持分级汇总，如查询每个兴趣的员工平均年龄:

```json
GET /megacorp/employee/_search
{
    "aggs" : {
        "all_interests" : {
            "terms" : { "field" : "interests" },
            "aggs" : {
                "avg_age" : {
                    "avg" : { "field" : "age" }
                }
            }
        }
    }
}
```

## 3. 分布式特性

Elasticsearch 尽可能地屏蔽了分布式系统的复杂性。这里列举了一些在后台自动执行的操作：

- 分配文档到不同的容器 或分片中，文档可以储存在一个或多个节点中
- 按集群节点来均衡分配这些分片，从而对索引和搜索过程进行负载均衡
- 复制每个分片以支持数据冗余，从而防止硬件故障导致的数据丢失
- 将集群中任一节点的请求路由到存有相关数据的节点
- 集群扩容时无缝整合新节点，重新分配分片以便从离群节点恢复

