---
title: ElasticSearch学习三
date: 2018-06-24 11:11:46
tags: [ElasticSearch,大数据]
---

## 1. 排序与相关性

### 1.1 排序

默认排序是按照相关性来排的，相关性得分由一个浮点数进行表示，并在搜索结果中通过 `_score` 参数返回， 默认排序是 `_score` 降序。

就像传统关系型数据库一样，ElasticSearch还可以排序，如我们想要查询user_id为1的用户发的tweet，按照时间降序排列：

```json
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : { "term" : { "user_id" : 1 }}
        }
    },
    "sort": { "date": { "order": "desc" }}
}
```

还可以多级排序：

```json
GET /_search
{
    "query" : {
        "bool" : {
            "must":   { "match": { "tweet": "manage text search" }},
            "filter" : { "term" : { "user_id" : 2 }}
        }
    },
    "sort": [
        { "date":   { "order": "desc" }},
        { "_score": { "order": "desc" }}
    ]
}
```

多值字段的排序：一个字段可能有多个值，对于数字或日期，你可以将多值字段减为单值，这可以通过使用 `min` 、 `max` 、 `avg` 或是 `sum` *排序模式* ：

```json
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```

### 1.2 如何计算相关性

每个文档都有相关性评分，用一个正浮点数字段 `_score` 来表示 。 `_score` 的评分越高，相关性越高。评分的计算方式取决于查询类型 不同的查询语句用于不同的目的： `fuzzy` 查询会计算与关键词的拼写相似程度，`terms` 查询会计算 找到的内容与关键词组成部分匹配的百分比，但是通常我们说的 *relevance* 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。

Elasticsearch 的相似度算法如下：

* 检索词频率

  检索词出现频率越高，相关性也越高

* 反向文档频率

  每个检索词在所有文档中出现的频率。频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。

* 字段长度准则

   检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段权重更大。

如果多条查询子句被合并为一条复合查询语句 ，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

## 2. 索引

### 2.1 创建索引

在第一片文章中就说了，索引有名词含义，这里的索引即指名词，意味着文档。目前我们创建索引都是采用的是默认的配置，新的字段通过动态映射的方式被添加到类型映射。如果我们要在建立索引的过程做更多的控制：如确保这个索引有数量适中的主分片，并且在我们索引任何数据之前 ，分析器和映射已经被建立好，就应该手动创建索引：

```json
PUT /my_index
{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }
}
```

### 2.2 删除索引

```json
DELETE /my_index
```

### 2.3 索引的设置

有两个比较重要的设置：

`number_of_shards`

每个索引的主分片数，默认值是 `5` 。这个配置在索引创建后不能修改。

`number_of_replicas`

每个主分片的副本数，默认值是 `1` 。对于活动的索引库，这个配置可以随时修改。

如，我们可以创建只有 一个主分片，没有副本的小索引：

```json
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}
```

也可以动态修改`number_of_replicas`:

```json
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
```

### 2.4 配置分析器

> 分析器,用于将全文字符串转换为适合搜索的倒排索引。

ElasticSearch应用于全文字段的是默认的`standard` 分析器，比较适合西语系，包括：

- `standard` 分词器，通过单词边界分割输入的文本。
- `standard` 语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）。
- `lowercase` 语汇单元过滤器，转换所有的语汇单元为小写。
- `stop` 语汇单元过滤器，删除停用词 -- 对搜索相关性影响不大的常用词，如 `a` ， `the` ， `and` ， `is`。

默认`stop` 语汇单元过滤器是金庸的，通过传递`stopwords` 参数可以启用它。下面的例子创建了一个新的分析器，使用预定义的西班牙stop word列表：

```json
PUT /spanish_docs
{
    "settings": {
        "analysis": {
            "analyzer": {
                "es_std": {
                    "type":      "standard",
                    "stopwords": "_spanish_"
                }
            }
        }
    }
}
```

### 2.5 创建自定义分析器

> 一个分析 就是在一个包里面组合了三种函数的一个包装器

三种函数就是指以下三种：

* 字符过滤器

  字符过滤器用来整理一个尚未被分词的字符串。比如讲HTML文本的HTML标签全部去掉以转化为一个纯字符串。

* 分词器

  分词器把字符串分解成单个词条或者词汇单元。

* 词单元过滤器

  文本通过分词器后成为词语流流向词单元过滤器。词单元过滤器可以修改、添加或者移除词单元。我们已经提到过`lowercase`和`stop` 词过滤器就是词单元过滤器。

那么所谓自定义分析器为非就是自己按照业务需求任意组合三种函数。格式如下：

```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}
```

如将&替换为and就使用这样的字符过滤器：

```json
"char_filter": {
    "&_to_and": {
        "type":       "mapping",
        "mappings": [ "&=> and "]
    }
}
```

定义一个词单元过滤器就是这样的

```json
"filter": {
    "my_stopwords": {
        "type":        "stop",
        "stopwords": [ "the", "a" ]
    }
}
```

合起来就是这样：

```json
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type":       "mapping",
                    "mappings": [ "&=> and "]
            }},
            "filter": {
                "my_stopwords": {
                    "type":       "stop",
                    "stopwords": [ "the", "a" ]
            }},
            "analyzer": {
                "my_analyzer": {
                    "type":         "custom",
                    "char_filter":  [ "html_strip", "&_to_and" ],
                    "tokenizer":    "standard",
                    "filter":       [ "lowercase", "my_stopwords" ]
            }}
}}}
```

告诉ElasticSearch在哪里用这个分析器（用在String上）：

```json
PUT /my_index/_mapping/my_type
{
    "properties": {
        "title": {
            "type":      "string",
            "analyzer":  "my_analyzer"
        }
    }
}
```