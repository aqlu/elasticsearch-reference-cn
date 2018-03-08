# Explain

开启执行计划可以查看每个命中文档是如何评分的。

```
GET /_search
{
    "explain": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```