# min_score

排除`_score`小于`min_score`中指定的最小值的文档：

```bash
GET /_search
{
    "min_score": 0.5,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

注意，大多数时候，这没有什么意义，但提供了高级用例。

 