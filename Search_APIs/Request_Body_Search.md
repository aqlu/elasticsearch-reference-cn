# Request Body Search（使用请求体进行搜索）

搜索请求能够通过DSL来执行，在请求体中包含这个[查询DSL](../Query_DSL.md)即可。下面是一个示例：

```js
GET /twitter/tweet/_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

这是一个示例的响应:

```js
{
    "took": 1,
    "timed_out": false,
    "_shards":{
        "total" : 1,
        "successful" : 1,
        "failed" : 0
    },
    "hits":{
        "total" : 1,
        "max_score": 1.3862944,
        "hits" : [
            {
                "_index" : "twitter",
                "_type" : "tweet",
                "_id" : "0",
                "_score": 1.3862944,
                "_source" : {
                    "user" : "kimchy",
                    "message": "trying out Elasticsearch",
                    "date" : "2009-11-15T14:12:12",
                    "likes" : 0
                }
            }
        ]
    }
}
```

## 参数

参数名                 | 描述
----------------------|-------------------
`timeout`             | 搜索超时，限制在指定时间值内执行的搜索请求，并在到期时收集的命中文档。默认为无超时。请参阅[时间单位](../API_Conventions/Common_options.md#time-units)一节。
`from`                | 从某个偏移量中检索匹配。默认为0。
`size`                | 要返回的命中数。默认为`10.`如果您不关心获取到的一些返回内容，但仅关注匹配或聚合的数量，将值设置为`0`将有助于提高性能。
`search_type`         | 要执行的搜索操作的类型。可以是`dfs_query_then_fetch`或`query_then_fetch`。默认为`query_then_fetch`。查看*搜索类型*了解更多。
`request_cache`       | 设置为`true`或`false`以启用或禁用在搜索请求`size`为`0`时的结果缓存，即聚合和建议（不返回顶部`hits`内容）。请参阅[Shard请求缓存](../Modules/Indices/Shard_request_cache.md)。
`terminate_after`     | 每个分片收集的最大文档数量，达到后查询执行将提前终止。如果设置，响应将有一个布尔字段`terminate_early`来指示查询执行是否实际已终止。默认为`terminate_after`。
`batched_reduce_size` | 分片结果的数量应该在协调节点上一次性减少。如果请求中潜在的分片数量可能较大，则该值应用作保护机制，以减少每个搜索请求的内存开销。

在上述中，`search_type`和`request_cache`必须作为查询字符串参数传递。搜索请求的其余部分应在主体本身内传递。请求体内容也可以作为名为`source`的REST参数传递。

`HTTP GET`和`HTTP POST`都可以用来执行与`body`的搜索。由于并非所有客户端都支持`GET`，所以`POST`也是允许的。

## 速检查任何匹配的文档

如果我们只想知道是否有匹配特定查询的文档，我们可以将大小设置为`0`，表示我们对搜索结果不感兴趣。此外，我们可以将`terminate_after`设置为`1`，以指示每当找到第一个匹配文档（每个分片）时，查询执行可以被终止。

```js
GET /_search?q=message:elasticsearch&size=0&terminate_after=1
```

响应不会包含大小设置为`0`的任何采样。`hits.total`等于`0`表示没有匹配的文档，或大于`0`表示至少有与查询匹配的数量的文档（当它被提前终止时）。此外，如果查询提前终止，则在响应中将`terminate_early`标志设置为`true`。

```js
{
  "took": 3,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0,
    "hits": []
  }
}
```