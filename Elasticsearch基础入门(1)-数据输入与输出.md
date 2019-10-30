---
title: Elasticsearch基础入门(1)-数据输入与输出
date: 2019-10-30 14:29:20
tags:
- Elastic Search
categories:
- Elastic Search
---

## 数据输入与输出
对象可以很好的表示现实世界中的实体，因此面向对象语言流行起来，但当我们需要存储这些实体是，传统上使用行和列存储数据到关系型数据库中，相当于电子表格，失去了对象的灵活性，那是否能用对象的方式存储对象呢？

对象是基于特定语言的数据结构，为了通过网络发送或存储它，需要把它变为标准格式，现在通常使用JSON，当一个对象被序列化成为JSON，它被称为一个JSON文档。

Elasticsearch是分布式的`文档`存储，能够以实时的方式存储和检索文档，在es中，每个字段的所有数据都是默认被索引的，即每个字段都有为了快速检索而设置的倒排索引，且能在同一个查询中使用所有这些倒排索引。

---
---
### 什么是文档
在大多数应用中，多数实体或对象可以被序列化为包含键值对的 JSON 对象。 一个 键 可以是一个字段或字段的名称，一个 值 可以是一个字符串，一个数字，一个布尔值， 另一个对象，一些数组值，或一些其它特殊类型诸如表示日期的字符串，或代表一个地理位置的对象：
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
通常情况下，我们使用的术语 对象 和 文档 是可以互相替换的。不过，有一个区别： 一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的 JSON 对象，对象中也可以嵌套其他的对象。 对象可能包含了另外一些对象。在 Elasticsearch 中，术语 文档 有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID。
___
### 文档元数据
一个文档不仅仅包含它的数据，也包含元数据——有关文档的信息。三个必须存在的元数据如下：
* _index: 文档在哪存放
* _type: 文档表示的对象类别
* _id: 文档唯一标识

#### _index
一个 索引 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。es中的索引有点像传统sql中数据库的概念。

注意索引名必须小写，不能以下划线开头，不能包含逗号

#### _type
数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" ，在我看来，这个`_type`和面向对象中`类`的概念相似。

一个 _type 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符. 

#### _id
ID 是一个字符串，当它和 _index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成。

#### 其他元数据
后面的章节讨论

---
### 索引文档
通过使用 index API ，文档可以被 索引 —— 存储和使文档可被搜索。 但是首先，我们要确定文档的位置。正如我们刚刚讨论的，一个文档的 _index 、 _type 和 _id 唯一标识一个文档。 我们可以提供自定义的 _id 值，或者让 index API 自动生成。
#### 使用自定义的ID
举个例子，如果我们的索引称为 website ，类型称为 blog ，并且选择 123 作为 ID ，那么索引请求应该是下面这样：
```json
PUT website/blog/123
{
    "title": "My first blog entry",
    "text":  "Just trying this out...",
    "date":  "2014/01/01"
}
```
返回的响应体如下
```json
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "123",
   "_version":  1,
   "created":   true
}
```
该响应表明文档已经成功创建，该索引包括 _index 、 _type 和 _id 元数据， 以及一个新元素： _version 。

在 Elasticsearch 中每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， _version 的值会递增。 在 处理冲突 中，我们讨论了怎样使用 _version 号码确保你的应用程序中的一部分修改不会覆盖另一部分所做的修改。
#### Autogenerating IDs
如果你的数据没有自然ID，es可以自动生成ID，注意不再使用`PUT`，而是使用`POST`
```json
POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}
```
除了 _id 是 Elasticsearch 自动生成的，响应的其他部分和前面的类似：
```json
{
   "_index":    "website",
   "_type":     "blog",
   "_id":       "AVFgSgVHUP18jI2wRx0w",
   "_version":  1,
   "created":   true
}
```
自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。

---
### 取回一个文档
为了从 Elasticsearch 中检索出文档，我们仍然使用相同的 _index , _type , 和 _id ，但是 HTTP 谓词更改为 GET ，这里pretty参数的是调用es的pretty-print美化输出结果，使响应体更可读，但是`_source`中的内容不能被美化。
```
GET /website/blog/123?pretty
```
响应体包括目前已经熟悉了的元数据元素，再加上 _source 字段，这个字段包含我们索引数据时发送给 Elasticsearch 的原始 JSON 文档：
```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "title": "My first blog entry",
      "text":  "Just trying this out...",
      "date":  "2014/01/01"
  }
}
```
GET 请求的响应体包括 `{"found": true}` ，这证实了文档已经被找到
#### 返回文档的一部分
默认情况下， GET 请求会返回整个文档，这个文档正如存储在 _source 字段中的一样。但是也许你只对其中的 title 字段感兴趣。单个字段能用 _source 参数请求得到，多个字段也能使用逗号分隔的列表来指定。
```json
GET /website/blog/123?_source=title,text
```
该 _source 字段现在包含的只是我们请求的那些字段，并且已经将 date 字段过滤掉了。
```json
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 1,
  "found" :   true,
  "_source" : {
      "title": "My first blog entry" ,
      "text":  "Just trying this out..."
  }
}
```
或者，如果你只想得到 _source 字段，不需要任何元数据，你能使用 _source 端点：
```
GET /website/blog/123/_source
```
那么返回的的内容如下所示：
```json
{
   "title": "My first blog entry",
   "text":  "Just trying this out...",
   "date":  "2014/01/01"
}
```
---
### 检查文档是否存在
用`HEAD`代替`GET`，HEAD请求无返回体，如果存在返回`200 ok`，不存在返回`404 not found`.

当然，一个文档仅仅是在检查的时候不存在，并不意味着一毫秒之后它也不存在：也许同时正好另一个进程就创建了该文档。

---
### 创建新文档
当我们索引一个文档，怎么确认我们正在创建一个完全新的文档，而不是覆盖现有的呢？

请记住， _index 、 _type 和 _id 的组合可以唯一标识一个文档。所以，确保创建一个新文档的最简单办法是，使用索引请求的 POST 形式让 Elasticsearch**自动**生成唯一 _id 

如果已经有自己的_id，那么我们必须告诉 Elasticsearch ，只有在相同的 _index 、 _type 和 _id 不存在时才接受我们的索引请求，有两种方法：
1. 使用 op_type 查询-字符串参数：
    ```
    PUT /website/blog/123?op_type=create
    { ... }
    ```
2. 在 URL 末端使用 /_create :
    ```
    PUT /website/blog/123/_create
    { ... }
    ```
如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 201 Created 的 HTTP 响应码。

另一方面，如果具有相同的 _index 、 _type 和 _id 的文档已经存在，Elasticsearch 将会返回 409 Conflict 响应码，以及错误信息

---
### 删除文档
删除文档的语法和我们所知道的规则相同，只是使用 DELETE 方法：
```
DELETE /website/blog/123
```
如果找到该文档，Elasticsearch 将要返回一个 200 ok 的 HTTP 响应码，和一个类似以下结构的响应体。注意，字段 _version 值已经增加:
```json
{
  "found" :    true,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 3
}
```
如果文档没有找到，我们将得到 404 Not Found 的响应码和类似这样的响应体：
```json
{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}
```
注意即使文档不存在（ Found 是 false ）， _version 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。