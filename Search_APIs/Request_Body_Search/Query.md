# Query

查询请求体中的`query`元素允许通过[Query DSL](../../Query_DSL.md)来定义一个查询。

```js
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```