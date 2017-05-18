# 搜索API

大多数搜索API是[多索引，多类型](./Search_APIs/Search.md#search-multi-index-type)，除了[Explain API](./Search_APIs/Explain_API.md)端点。

## 路由

当执行搜索时，它将被广播到所有索引/索引分片（副本间轮询）。可以通过提供路由参数来控制哪些分片将被搜索。例如，索引`tweets`时，路由值可以是用户名：

```js
POST /twitter/tweet?routing=kimchy
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在这种情况下，如果要仅搜索特定用户的推文，我们可以将其指定为路由，导致搜索仅触发相关的分片：

```js
POST /twitter/tweet/_search?routing=kimchy
{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
```

路由参数可以是多值，以逗号分隔的字符串表示。这将导致与路由值匹配的相关分片被命中。

## 统计群组

搜索可以与统计组相关联，维护每个组的统计信息聚合。稍后可以使用[indices stats](./Indices_APIs/Indices_Stats.md) API进行检索。例如，这是一个搜索请求主体，将请求与两个不同的组相关联：

```js
POST /_search
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
```

## 全局搜索超时

个体搜索可以将超时作为[搜索请求正文](./Search_APIs/Request_Body_Search.md)的一部分。由于搜索请求可能来自许多来源，Elasticsearch具有群集级别的全局搜索超时的动态设置，适用于在[搜索请求正文](./Search_APIs/Request_Body_Search.md)中未设置超时的所有搜索请求。默认值没有全局超时。设置键为`search.default_search_timeout`，可以使用[群集更新设置](./Cluster_APIs/Cluster_Update_Settings.md)端点进行设置。将此值设置为`-1`会将全局搜索超时重置为无超时。

## 搜索取消

可以使用标准[任务取消](./Cluster_APIs/Task_Management_API.md#task-cancellation)机制取消搜索。默认情况下，正在运行的搜索仅检查段边界是否被取消，因此取消可以被大段延迟。通过集群级别设置动态将`search.low_level_cancellation`设置为`true`可以提高搜索取消响应度。然而，它带来了更频繁的取消检查的额外开销，这在大型快速运行的搜索查询上可以是明显的。更改此设置仅影响在进行更改后开始的搜索。
