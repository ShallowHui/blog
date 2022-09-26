---
title: Elasticsearch入门
date: 2022-8-26 16:29:11
tags: Elasticsearch
categories: Java
cover: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/elasticsearch.jpg
top_img: https://cdn.jsdelivr.net/gh/shallowhui/cdn/picgo/elasticsearch.jpg
description: 一款Java开发的，分布式的，可以对存储的数据进行搜索、分析的引擎。
---
## Elasticsearch介绍

在其[官网](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/elasticsearch-intro.html)中，有如下介绍：

>Elasticsearch是位于Elastic Stack核心的分布式搜索和分析引擎。Logstash和Beats有助于收集、聚合和丰富您的数据并将其存储在Elasticsearch中。Kibana使您能够以交互方式探索、可视化和分享对数据的见解，并管理和监控Stack。Elasticsearch是进行索引、搜索和分析的地方。
>
>Elasticsearch为所有类型的数据提供近乎实时的搜索和分析。无论您拥有结构化或非结构化文本、数字数据还是地理空间数据，Elasticsearch都能以支持快速搜索的方式高效地存储和索引它。您可以进行数据检索和聚合信息来发现数据中的趋势和模式。随着您的数据和查询量的增长，Elasticsearch的分布式特性使您的部署能够随之无缝增长。

显然，Elasticsearch不仅可以存储数据，还可以对数据进行`近实时`的搜索和分析，正如它的口号：`You konw, for search`。

不同于传统的关系型数据库，通过SQL对结构化数据进行查询（搜索），Elasticsearch通过DSL（特定领域语言），不仅可以对结构化数据进行查询，比如匹配查询、范围查询，还可以对非结构化的文本数据进行全文搜索，这是关系型数据库很难完成的任务。

存储到Elasticsearch中的数据，会近乎实时地（1秒内）被建立索引然后就完全可以对其进行搜索。**这是因为Elasticsearch会为每一个字段建立索引，每个被索引的字段都有一个专用的优化数据结构，例如，文本（text）字段存储在`倒排索引`中，而数字和地理字段存储到`BKD`树中。**

  Elasticsearch会为文档中的每一个`text`类型的字段建立倒排索引，通过倒排索引，可以找到包含某个关键词（Term）的所有文档（Posting List）。

Elasticsearch是基于Apache旗下的`Lucene`进行开发的，Lucene才是真正工作的搜索和分析引擎，Elasticsearch在其之上做了个封装，隐藏其复杂性，主要负责管理Elasticsearch集群，并对外提供基于HTTP的RESTful接口。

这篇文章简单介绍一下`8.x`版本的Elasticsearch。

## 下载启动

下面是下载页面的链接，下载按钮后面还跟有Elasticsearch的启动步骤：

[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)

**注意，较新版本的Elasticsearch都是默认启用security的，要注意命令行启动Elasticsearch时输出的信息：**

```bash
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------> Elasticsearch security features have been automatically configured!
-> Authentication is enabled and cluster connections are encrypted.

->  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  CFhKpGiQiKtcYLnzKVAT

->  HTTP CA certificate SHA-256 fingerprint:
  9e890980d426fb3ed8a55faf1314a30d88713c339c4d3eabb48893dedfc81bae

->  Configure Kibana to use this cluster:
* Run Kibana and click the configuration link in the terminal when Kibana starts.
* Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjMuMyIsImFkciI6WyIxNzIuMTYuMTMuMjk6OTIwMCJdLCJmZ3IiOiI5ZTg5MDk4MGQ0MjZmYjNlZDhhNTVmYWYxMzE0YTMwZDg4NzEzYzMzOWM0ZDNlYWJiNDg4OTNkZWRmYzgxYmFlIiwia2V5IjoiQzhYdDA0SUJoaTR1eHNnU0l0ajY6UHhGLS0tOVNUcS1ybk1kY3F1ZHpldyJ9

->  Configure other nodes to join this cluster:
* On this node:
  - Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  - Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  - Restart Elasticsearch.
* On other nodes:
  - Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

由于开启了security，所以通过RESTful接口（默认端口是`9200`）访问Elasticsearch时，协议必须是`HTTPS`，并且访问之前要通过身份验证。默认内建的超级账号名为`elastic`，密码可以在启动时输出的信息中找到。

在集群中，Elasticsearch通过TCP与其它Elasticsearch节点进行通信，默认端口为`9300`。

## 数据存储

Elasticsearch可以用来存储数据，但与传统关系型数据库不同的是，它存储数据不是按行列进行存储，而是存储`文档`。

### 文档

其实，这里说的文档就是一个JSON对象，Elasticsearch使用JavaScript Object Notation（JSON）作为文档的序列化格式。JSON序列化为大多数编程语言所支持，并且已经成为`NoSQL`领域的标准格式：

```json
{
    "email":      "zunhuier@google.com",
    "first_name": "",
    "last_name":  "",
    "info": {
        "age":         26,
        "interests": [ "", "" ]
    },
    "join_date": ""
}
```

一个文档由多个`filed`（字段）组成，正如前面所说，一旦文档被存储到Elasticsearch中，Elasticsearch默认就会为文档的每一个字段建立相应的索引，并且可以自动识别不同的数据类型。

### 索引

以关系型数据库作为对比，那么文档就相当于表中的一条数据记录，而索引就相当于数据库。本来中间是还有一个`type`（类型），相当于数据表的，但Elasticsearch从8开始弃用type这个概念。

这里可能对索引的概念的有点迷糊，需要对其在不同语境下的含义进行明确：

1. 作名词：如前所述，相当于关系型数据库中的数据库概念，指Elasticsearch中，**具有大致相同结构和含义的一组文档的逻辑集合**。

2. 作动词：将文档存储到Elasticsearch中，也可以表述为：将文档索引到Elasticsearch中，因为Elasticsearch马上就会为文档建立索引。

3. 作名词：为文档建立索引，这里的索引就是指那些索引结构了，比如倒排索引。

## RESTful接口

简单介绍一下通过RESTful接口与Elasticsearch进行交互。

### 添加删除索引

添加索引：

`PUT /{index}`

```
PUT /test

返回结果：
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test"
}
```

删除索引：

`DELETE /{index}`

```
DELETE /test

返回结果：
{
  "acknowledged": true
}
```

### 添加删除文档

添加文档，通过在请求体中携带JSON数据：

`POST /{index}/_create/{id}`

```
POST /test/_create/1
{
  "name": "zunhuier",
  "age": 26
}

返回结果：
{
  "_index": "test",
  "_id": "1",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}
```

路径参数中的`test`是指索引，如果不存在，Elasticsearch会自动创建。`1`用于指定文档的ID，如果不指定，Elasticsearch会为文档自动生成一个ID。

删除文档：

`DELETE /{index}/_doc/{id}`

```
DELETE /test/_doc/1

返回结果：
{
  "_index": "test",
  "_id": "1",
  "_version": 2,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 8,
  "_primary_term": 1
}
```

### 查看索引

`GET /{index}`

```
GET /test

返回结果：
{
  "test": {
    "aliases": {},
    "mappings": {
      "properties": {
        "age": {
          "type": "long"
        },
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        }
      }
    },
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "include": {
              "_tier_preference": "data_content"
            }
          }
        },
        "number_of_shards": "1",
        "provided_name": "test",
        "creation_date": "1661419717653",
        "number_of_replicas": "1",
        "uuid": "0m_-NwEpQfqTeaa259yg4g",
        "version": {
          "created": "8030399"
        }
      }
    }
  }
}
```

### 查看文档

`GET /{index}/_doc/{id}`

```
GET /test/_doc/1

返回结果：
{
  "_index": "test",
  "_id": "1",
  "_version": 1,
  "_seq_no": 21,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "zunhuier",
    "age": 26
  }
}
```