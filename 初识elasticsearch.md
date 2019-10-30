---
title: 初识elasticsearch
date: 2019-10-30 11:30:10
tags:
- Elastic Search
categories:
- Elastic Search
---

用一个例子来解释es中**索引(index)，文档(document)，类型(type)**的概念。
第一个业务需求是存储员工数据。 这将会以 员工文档 的形式存储：一个文档代表一个员工。存储数据到 Elasticsearch 的行为叫做 索引 ，但在索引一个文档之前，需要确定将文档存储在哪里。

一个 Elasticsearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性 。
索引在es中有多种语义
* 索引（名词）： 一个索引类似于关系型数据库中的一个数据库，是存储关系型文档的地方，复数为indices
* 索引（动词）：索引一个文档就是存储一个文档到一个索引，类似于sql中的insert
* 文档：是es中的主要实体，包含多个字段，每个字段有字段名和字段值

### 索引文档
对于员工目录，我们将做如下操作：
1. 每个员工索引一个文档，文档包含该员工的所有信息。
2. 每个文档都将是 employee 类型 。
3. 该类型位于 索引 megacorp 内。
4. 该索引保存在我们的 Elasticsearch 集群中。

可以通过以下命令完成
```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}
```
注意，路径 /megacorp/employee/1 包含了三部分的信息：

**megacorp**:索引名称

**employee**:类型名称

**1**:特定雇员的ID



### 检索文档
执行HTTP GET请求，指定文档的地址——索引库+类型+ID
```json
GET /megacorp/employee/1
```
结果返回
```json
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```
同样的，可以使用DELETE 删除文档，PUT可以添加/更新已存在的文档。

### 轻量搜索
使用以下请求搜索所有雇员，这次的返回结果包括所有文档，放在hits数组中。
```
GET /megacorp/employee/_search
```

使用查询字符串（query-string）搜索，搜索姓氏为‘smith’的雇员
```
GET /megacorp/employee/_search?q=last_name:Smith
```

### 使用查询表达式
使用领域特定语言（DSL），使用json构造了一个请求，重写之前的查询所有名为 Smith 的搜索 ：
```JSON
GET //employee/_serach
{
    "query": {
        ”match": {
            "last_name": "Smith"
        }
    }
}
```
此时不在使用query-string参数，而是用一个请求体代替,使用了一个match查询

### 全文搜索
搜索下所有喜欢攀岩（rock climbing）的员工：
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
返回结果如下：
```json
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, 
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, 
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```
可以看到一项参数`_score`，这是参数是相关性得分，es默认按照相关性得分排序，这是区别于传统数据库的一点，传统数据库中一条记录要么匹配要不不匹配

### 短语搜索
当需要精确匹配一系列单词或短语， 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

为此对 match 查询稍作调整，使用一个叫做 match_phrase 的查询：
```json
GET /megacorp/employee/_search
{
    "query": {
        "match_phrase": {
            "about":"rock climbing"
        }
    }
}
```
此时，只返回如下一条记录
```json
{
   ...
   "hits": {
      "total":      1,
      "max_score":  0.23013961,
      "hits": [
         {
            ...
            "_score":         0.23013961,
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         }
      ]
   }
}
```

### 高亮搜索
使匹配部分高亮
```json
GET /megacorp/employee/_search
{
    "query": {
        "match": {
            "about": "rock climbing"
        }
    },
    "highlight": {
      "fields": {
        "about": {}
      }
    }
}
```
在返回结果的hits字段中多了一个highlight字段，这个字段包含了匹配的文本片段，加了`<em></em>`封装

