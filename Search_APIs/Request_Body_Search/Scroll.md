# Scroll

虽然`search`请求返回单个“page”的结果，但是`Scroll`API 可以用于从单个搜索请求中检索大量结果（甚至是所有结果），与在传统数据库上使用游标的方式大致相同。

滚动不是用于实时用户请求，而是用于处理大量数据，例如，以便将一个索引库的内容重新索引到具有不同配置的新索引库中。


> ## 支持滚动和重建索引的客户端
>
> 一些官方支持的客户端提供帮手以协助从一个索引到另一个索引的滚动搜索和重新索引文档：
>
> - Perl
>     - 请参见: [Search::Elasticsearch::Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk)和[Search::Elasticsearch::Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)
>
> - Python
>   - 请参见：[elasticsearch.helpers.*](http://elasticsearch-py.readthedocs.org/en/master/helpers.html)


> 注意：
>
> 从滚动请求返回的结果反映了进行初始搜索请求时索引的状态，就像在此时间点的快照。对文档（索引，更新或删除）的后续更改只会影响以后的搜索请求。


为了使用滚动，初始搜索请求应该在查询字符串中指定滚动参数，告诉Elasticsearch应保持“搜索上下文”活动的时间（见[ Keeping the search context alive](#keeping_the_search_context_alive)），例如 `?scroll=1m`。

```bash
POST /twitter/tweet/_search?scroll=1m
{
    "size": 100,
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
```

上述请求的结果包括一个`_scroll_id`，它应该被传递给`scroll`API，以便检索下一批结果。

```bash
POST ①/_search/scroll ②
{
    "scroll" : "1m", ③
    "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==" ④
}
```

① 可以使用GET或POST。
____________________________
② 网址不应包含索引或类型名称 - 而是在原始搜索请求中指定的。
____________________________
③ scroll 参数告诉 Elasticsearch 将搜索上下文打开另一个1m。
____________________________
④ scroll_id参数
____________________________

`size`参数允许您配置每批结果返回的最大命中数。 每次调用 scroll API 都会返回下一批结果，直到没有更多结果要返回，即`hits`数组为空。

> ##重要：##
>
> 初始搜索请求和每个后续滚动请求返回一个新的`_scroll_id` —— 只应该使用最近的`_scroll_id`。

> 注意
>
> 如果请求指定了聚合，则只有初始搜索响应将包含聚合结果。

> 注意
>
> 滚动请求具有优化，使排序顺序为`_doc`时更快。 如果你想迭代所有文档，无论顺序如何，这是最有效的选择：

```bahs
GET /_search?scroll=1m
{
  "sort": [
    "_doc"
  ]
}
```

## <span id="keeping_the_search_context_alive">Keeping the search context alive</span>

滚动参数（传递到搜索请求和每个滚动请求）告诉Elasticsearch应该保持搜索上下文活动的时间。其值（例如，`1m`，请参阅[Time unit](../../API_Conventions/Common_options.md#time-units/)一节）不需要足够长以处理所有数据 —— 它只需要足够长的时间来处理前一批结果。每个滚动请求（具有滚动参数）设置新的到期时间。

通常，后台合并过程通过将较小的段合并在一起以创建新的较大段来优化索引，此时较小的段被删除。此过程在滚动期间继续，但是打开的搜索上下文防止旧段在它们仍在使用时被删除。这就是Elasticsearch如何能够返回初始搜索请求的结果，而不考虑对文档的后续更改。

> 提示：
>
> 保持较旧的段仍然活动意味着需要更多的文件句柄。确保您已将节点配置为具有足够的可用文件句柄。请参阅[File Descriptors](../../Setup_Elasticsearch/Important_System_Configuration/File_Descriptors.md)。

您可以使用[nodes stats API](../../Cluster_APIs/Nodes_Stats.md)检查打开的搜索数量：

```bash
GET /_nodes/stats/indices/search
```

## Clear scroll API

当超出`scroll`超时时，将自动删除搜索上下文。 但是，保持滚动打开有一个成本，如[前面部分](#keeping_the_search_context_alive)所讨论的，因此一旦`scroll`不再使用，应该明确清除，clear-scroll API：

```bash
DELETE /_search/scroll
{
    "scroll_id" : ["DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ
=="]
}
```

多个滚动 ID 可以作为数组传递：

```bash
DELETE /_search/scroll
{
    "scroll_id" : [
      "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==",
      "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAABFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAAAxZ
rUllkUVlCa1NqNmRMaUhiQlZkMWFBAAAAAAAAAAIWa1JZZFFZQmtTajZkTGlIYkJWZDFhQQAAAAAAAAAFF
mtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAABBZrUllkUVlCa1NqNmRMaUhiQlZkMWFB"
    ]
}
```

所有搜索上下文可以使用`_all`参数清除：

```bash
DELETE /_search/scroll/_all
```

`scroll_id`也可以作为查询字符串参数或在请求正文中传递。 多个滚动 ID 可以作为逗号分隔值传递：

```bash
DELETE /_search/scroll/DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ==,DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAAABFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAAAxZrUllkUVlCa1NqNmRMaUhiQlZkMWFBAAAAAAAAAAIWa1JZZFFZQmtTajZkTGlIYkJWZDFhQQAAAAAAAAAFFmtSWWRRWUJrU2o2ZExpSGJCVmQxYUEAAAAAAAAABBZrUllkUVlCa1NqNmRMaUhiQlZkMWFB
```

## <span id="sliced-scroll">滚动切片</span>

对于返回大量文档的滚动查询，可以在能独立使用的多个切片中拆分滚动：

```bash
GET /twitter/tweet/_search?scroll=1m
{
    "slice": {
        "id": 0,        # ①
        "max": 2     # ②
    },
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
GET /twitter/tweet/_search?scroll=1m
{
    "slice": {
        "id": 1,
        "max": 2
    },
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
```

① 切片 ID
______________
② 最大切片数
______________

第一个请求的结果返回属于第一个切片（id：0）的文档，第二个请求的结果返回属于第二个切片的文档。 由于切片的最大数目被设置为2，所以两个请求的结果的并集相当于没有切片的滚动查询的结果。 默认情况下，首先在分片上进行分割，然后使用带有以下公式的`_uid`字段在每个分片上进行本地分割：`slice(doc)= floorMod(hashCode(doc._uid)，max)`例如，如果分片数为`2`并且用户请求`4`个切片，则将片`0`和`2`分配给第一分片，并且将片`1`和`3`分配给第二分片。

每个滚动是独立的，并且可以像任何滚动请求一样被并行处理。

> 注意：
>
> 如果片的数量大于分片的数量，则片滤波器在第一次调用时非常慢，其具有O（N）的复杂度，并且存储器成本等于每片N比特，其中N是文档的总数 在碎片中。 在少数调用之后，过滤器应该被缓存，并且随后的调用应当更快，但是应该限制并行执行的分片查询的数量，以避免内存爆炸。

为了完全避免此成本，可以使用另一个字段的`doc_values`进行切片，但用户必须确保该字段具有以下属性：

- 该字段是数字。
- 在该字段上启用了 doc_values
- 每个文档应包含单个值。 如果文档具有指定字段的多个值，则使用第一个值。
- 在创建文档并且从不更新时，应为每个文档设置一次。 这确保每个切片获得确定性结果。
- 字段的基数应该很高。 这确保每个切片获得大致相同量的文档。

```bash
GET /twitter/tweet/_search?scroll=1m
{
    "slice": {
        "field": "date",
        "id": 0,
        "max": 10
    },
    "query": {
        "match" : {
            "title" : "elasticsearch"
        }
    }
}
```

对于仅追加基于时间的索引，可以安全地使用时间戳字段。

> 注意：
>
> 默认情况下，每个滚动允许的最大切片数限制为`1024`。您可以更新 `index.max_slices_per_scroll`索引设置以绕过此限制。

