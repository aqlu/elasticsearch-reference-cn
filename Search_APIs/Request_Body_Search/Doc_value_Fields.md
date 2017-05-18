# Doc value Fields

允许返回每个匹配的字段的[doc value](../..//Mapping/Mapping_parameters/doc_values.md)表示形式，例如：

```js
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "docvalue_fields" : ["test1", "test2"]
}
```

Doc value Fields 可以用于未存储的字段。

请注意，如果`fields`参数指定没有`docvalues`的字段，它将尝试从`fielddata`缓存加载值，使得该字段的词条被加载到内存（缓存），这将导致更多的内存消耗。