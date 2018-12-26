---
title: ElasticSearch学习二-文档与搜索.md
date: 2018-06-22 15:15:15
tags: [大数据,ElasticSearch]
---

## 1. 文档

在 Elasticsearch 中，每个字段的所有数据都是默认被索引的。

大多数实体和对象都可以被序列化为包含键值对的JSON对象，JSON对象能表示字符串、数字、布尔值、甚至另一个对象。如下所示：

```json
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```

> 文档的定义：它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。

### 1.1 文档的元数据

> 元数据 —— 有关文档的信息,与文档包含的数据相区分。

有如下三个元数据：

1. _index:文档存放在哪

   一个索引应该是因共同的特性被分组到一起的文档集合，如可能存储所有的产品在索引 `products` 中。索引名必须小写，不能以下划线开头，不能包含逗号。

2. _type:文档的类型

   Elasticsearch 公开了一个称为 *types* （类型）的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。相当于在_index下继续细分。

3. _id:文档的唯一标识

   *ID* 是一个字符串， 当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。 

### 1.2 储存文档

如何确定一个文档的位置呢？就如上面所说的，三个元数据确定一个文档。如储存一个文档：

```json
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

 如何像数据库自增主键那样由ElasticSearch提供id呢，使用POST谓词就可以了。

```json
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```

会自动生成一个基于 Base64 编码且长度为 20 个字符的 GUID 字符串

### 1.3 取回文档

同样的只需要把谓词改为GET，同时提供三个元数据就可以了：

```http
GET /website/blog/123?pretty
```

加上pretty参数使得 JSON 响应体更加可读。如果只想返回文档的一部分则在url上添加_source指定想要的字段就行了。

### 1.4 更新整个文档

在 Elasticsearch 中文档是不可改变的，不能修改它们，只能进行替换。相当于使用PUT再重新将整个文档PUT一下，Elasticsearch将旧文档标记为已删除，并增加一个全新的文档。

（创建和删除文档略）

### 1.5 部分更新文档

使用 `update` API可以部分更新文档，例如在某个请求时对计数器进行累加。对象被合并到一起，覆盖现有的字段，增加新的字段。如：

```json
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

在文档中增加了tags和views字段。

也可以使用脚本和参数更新：

```json
POST /website/blog/1/_update
{
   "script" : "ctx._source.tags+=new_tag",
   "params" : {
      "new_tag" : "search"
   }
}
```

为了避免数据丢失， `update` API 在检索步骤时检索得到文档当前的 `_version` 号，并传递版本号到重建索引步骤的 `index` 请求。 如果另一个进程修改了处于检索和重新索引步骤之间的文档，那么 `_version` 号将不匹配，更新请求将会失败。

还可以在url后加上`retry_on_conflict=5`参数，表示失败之前重试5次。

### 1.6 乐观并发控制

ElasticSearch使用乐观并发控制，每个文档都有一个版本号\_version，当文档被修改时版本号递增。 Elasticsearch 使用这个 `_version` 号来确保变更以正确顺序得到执行。我们通过指定想要修改文档的 `version` 号来达到这个目的，如果该版本不是当前版本号，我们的请求将会失败（也就是一次Compare And Set）。

### 1.7 批量取回多个文档

使用mget API来取回多个文档可以减少网络流量，参数在请求体里的docs数组中提供：

```json
GET /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```

 还有其他的批量查询方法可以查看[官方文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Retrieving_Multiple_Documents.html)。

返回文档顺序和请求的一致。

### 1.8 批量操作

很简单一看就懂，使用bulk API，主要就是语法的问题，不太好掌握。

```json
POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} } 
```



## 2. 搜索 

### 2.1 空搜索

什么条件也不附加的空搜索，搜索ElasticSearch中的所有文档：

```http
GET /_search
```

返回所有查询结果的前十个文档而不是只是返回id。

### 2.2 指定index，type搜索

前面的空搜索是搜索所有文档，如果要想指定元数据搜索怎么办呢？很好办，在url中指明即可：

`/gb/_search`

在 `gb` 索引中搜索所有的类型

`/gb,us/_search`

在 `gb` 和 `us` 索引中搜索所有的文档

`/g*,u*/_search`

在任何以 `g` 或者 `u` 开头的索引中搜索所有的类型

`/gb/user/_search`

在 `gb` 索引中搜索 `user` 类型

`/gb,us/user,tweet/_search`

在 `gb` 和 `us` 索引中搜索 `user` 和 `tweet` 类型

`/_all/user,tweet/_search`

在所有的索引中搜索 `user` 和 `tweet` 类型

### 2.3 分页

和传统的数据库一样，ElasticSearch也有分页的功能，使用`from` 和 `size` 参数：

`size`

显示应该返回的结果数量，默认是 `10`

`from`

显示应该跳过的初始结果数量，默认是 `0`

```http
GET /_search?size=5&from=10
```

### 2.4 轻量级搜索

之前我们看到的搜索都是有个JSON请求体，但是还有一种更简便的轻量级搜索方式.

1. 搜索所有包含关键字的文档：

   ```http
   GET /_search?q=mary
   ```

2. 搜索包含关键字的所有文档：

   ```http
   GET /_search?q=mary
   ```

3. 指定匹配多个关键字的文档：

   ```http
   GET /_search?q=+name:john +tweet:mary
   ```

   `+` 前缀表示必须与查询条件匹配。类似地， `-` 前缀表示一定不与查询条件匹配。

   注意+、-、：都要使用URL编码，实际上上面的查询语句应该是：

   ```http
   GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
   ```

那么ElasticSearch如何从JSON的不同字段搜索关键字呢？原来ElasticSearch在储存一个文档上，会将该JSON所有字段提取出来拼接成一个字符串，然后增加一个_all字段，将该字符串添加到all字段上。

## 3. 请求体查询

请求体查询就像是一个带请求体的GET请求：

```http
GET /_search
{
  "from": 30,
  "size": 10
}
```

### 3.1 查询表达式

Elasticsearch 使用查询表达式可以以简单的 JSON 接口来编写查询语句。只需要将语句传递给query参数就可以了。如查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：

```json
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

当你有多个查询条件是可以合并查询语句：

```json
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}
```

### 3.2 重要的查询

1. 默认查询`match_all`匹配所有文档：

```json
{ "match_all": {}}
```

2. 标准查询match，可用于全文搜索和精确查询：

```json
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

3. mulit_match查询，可以在多个字段上执行相同的 `match` 查询：

```json
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

4. range查询，有gt,gte,lt,lte字段:

```json
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

5. term查询用于精确匹配，不对查询字符串做分析处理

```json
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

### 3.3 组合多查询

如果想在多个字段上查询多种多样的文本怎么办呢？可以使用`bool` 查询来实现你的需求。`bool` 查询接收以下参数：

`must`

文档 *必须* 匹配这些条件才能被包含进来。

`must_not`

文档 *必须不* 匹配这些条件才能被包含进来。

`should`

如果满足这些语句中的任意语句，将增加 `_score` ，否则，无任何影响。它们主要用于修正每个文档的相关性得分。

`filter`

*必须* 匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

如下面这个查询的意思是必须满足`title` 字段匹配 `how to make millions`并且不被标识为 `spam` 的文档。如果是starred且在2014年后的邮件会有更高的排名：

```json
"bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
```

每一个子查询都独自地计算文档的相关性得分。一旦他们的得分被计算出来， `bool` 查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

bool查询默认是评分的，将 `bool` 查询包裹在 `filter` 语句中，就可以在过滤标准中增加布尔逻辑。