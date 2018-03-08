# Named Queries

每个过滤器和查询可以在其顶级定义中接受一个`_name`。

```bash
GET /_search
{
    "query": {
        "bool" : {
            "should" : [
                {"match" : { "name.first" : {"query" : "shay", "_name" : "first"} }},
                {"match" : { "name.last" : {"query" : "banon", "_name" : "last"} }}
            ],
            "filter" : {
                "terms" : {
                    "name.last" : ["banon", "kimchy"],
                    "_name" : "test"
                }
            }
        }
    }
}
```

搜索响应将为每个匹配项添加其匹配的`matched_queries`。 查询和过滤器的标记仅对`bool`查询有意义。
