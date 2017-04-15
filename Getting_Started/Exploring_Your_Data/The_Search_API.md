# 搜索API

现在，让我们以一些简单的搜索来开始学习。有两种基本的方式来运行搜索：一种是在REST请求的URI中发送搜索参数，另一种是将搜索参数发送到REST请求体中。请求体方法的表达能力更好，并且你可以使用更加可读的JSON格式来定义搜索。我们将尝试使用一次请求URI作为例子，但是教程的后面部分，我们将仅仅使用请求体方法。

搜索的REST API可以通过`_search`端点(endpoint)来访问。下面这个例子返回`bank`索引中的所有的文档：

```js
GET /bank/_search?q=*&sort=account_number:asc&pretty
```

先来分析一下搜索请求。我们在`bank`索引库中搜索（ `_search`端点），并且`q=*`参数指示Elasticsearch去匹配这个索引中所有的文档。`sort=account_number:asc`指示结果按`account_number`字段升序排列。pretty参数仅仅是告诉Elasticsearch返回美观的JSON结果。

以下是响应（部分列出）：

```js
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

对于这个响应，我们可以看到如下的部分：

- `took`：Elasticsearch 执行这个搜索的耗时，以毫秒为单位
- `timed_out`：指明这个搜索是否超时
- `_shards`：指出多少个分片被搜索了，同时也标记了搜索成功与失败分片的数量
- `hits`：搜索结果
- `hits.total`：匹配查询条件的文档的总数目
- `hits.hits`：真正的搜索结果数组（默认是前10个文档）
- `hits.sort`：结果排序字段（如果缺失则按照得分排序）
- `hits._score` 和 `max_score`：现在先忽略这些字段

使用请求体方法的等价搜索是：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

与上面方法不同之处在于，并不是向URI中传递`q=*`，取而代之的是在`_search` API的请求体中POST了一个JSON格式的请求体。我们将在下一个章节中讨论这个JSON查询。

有一点你需要重点关注，一旦你取回了搜索结果，Elasticsearch就完成了使命，它不会保持任何服务器端的资源或者在你的结果中打开游标。这是和其它像SQL平台的一个鲜明的对比， 在那些平台上，你在先获取到查询结果的子集之后可以不断的从服务器提取结果中的剩余部分，这个数据集使用了一种有状态的服务器端游标技术。