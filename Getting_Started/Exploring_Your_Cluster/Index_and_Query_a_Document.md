# 索引文档创建与查询

现在让我们放一些东西到`customer`索引库中。首先要知道的是，要创建一个索引文档，我们必须要告诉Elasticsearch这个文档要存放到这个索引库的哪个类型（type）下。

让我们将一个简单的会员文档索引到`customer`索引库的“external”类型中，这个文档的ID是1，操作如下：

```js
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
```

响应如下：

```js
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "created" : true
}
```

从上面的响应中，我们可以看到，一个新的会员文档在customer索引库的external类型中被成功创建。文档也有一个内部id 1， 这个id是我们在创建索引文档的时候指定的。

需要注意的是，当你想将文档索引到某个索引库时候，Elasticsearch并不强制要求你先显式地创建索引库。在前面这个例子中，如果`customer`索引库不存在，Elasticsearch将会自动地创建这个索引。

现在，让我们把刚刚索引的文档取出来：

```js
GET /customer/external/1?pretty
```

响应如下：

```js
{
  "_index" : "customer",
  "_type" : "external",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

没有什么特别的，除了`found`字段标识我们找到这个ID为1的文档外，_source字段采用JSON格式返回了我们之前创建的完整文档。