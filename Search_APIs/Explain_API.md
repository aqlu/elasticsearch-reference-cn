# Explain API（执行计划API）

执行计划API能对特定文档查询评分进行说明。这可以给出文档是否与特定查询匹配的有用反馈。

`index`和`type`参数分别期望单个索引和单个类型。

## 使用方式

完整查询示例：

```js
GET /twitter/tweet/0/_explain
{
      "query" : {
        "match" : { "message" : "elasticsearch" }
      }
}
```

这将产生以下结果：

```js
{
   "_index": "twitter",
   "_type": "tweet",
   "_id": "0",
   "matched": true,
   "explanation": {
      "value": 1.55077,
      "description": "weight(message:elasticsearch in 0) [PerFieldSimilarity], result of:",
      "details": [
         {
            "value": 1.55077,
            "description": "score(doc=0,freq=1.0 = termFreq=1.0\n), product of:",
            "details": [
               {
                  "value": 1.3862944,
                  "description": "idf, computed as log(1 + (docCount - docFreq + 0.5) / (docFreq + 0.5)) from:",
                  "details": [
                     {
                        "value": 1.0,
                        "description": "docFreq",
                        "details": []
                     },
                     {
                        "value": 5.0,
                        "description": "docCount",
                        "details": []
                      }
                   ]
               },
                {
                  "value": 1.1186441,
                  "description": "tfNorm, computed as (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * fieldLength / avgFieldLength)) from:",
                  "details": [
                     {
                        "value": 1.0,
                        "description": "termFreq=1.0",
                        "details": []
                     },
                     {
                        "value": 1.2,
                        "description": "parameter k1",
                        "details": []
                     },
                     {
                        "value": 0.75,
                        "description": "parameter b",
                        "details": []
                     },
                     {
                        "value": 5.4,
                        "description": "avgFieldLength",
                        "details": []
                     },
                     {
                        "value": 4.0,
                        "description": "fieldLength",
                        "details": []
                     }
                  ]
               }
            ]
         }
      ]
   }
}
```

还有一种通过`q`参数指定查询的更简单的方法。然后解析指定的`q`参数值，就像使用`query_string`查询一样。在 explain api 中使用q参数的示例：

```js
GET /twitter/tweet/0/_explain?q=message:search
```

这将产生与先前请求相同的结果。

所有参数：

参数名              | 描述
-------------------|----------------
`_source`          | 设置为`true`以提取文档的`_source`。您还可以使用`_source_include`＆ `_source_exclude`检索文档的一部分（有关详细信息，请参阅[Get API](../Document_APIs/Get_API.md)）
`stored_fields`    |允许控制哪些存储字段作为文档的一部分返回。
`routing`          | 在创建索引期间使用路由的情况下控制路由。
`parent`           | 与设置路由参数相同的效果。
`preference`       | 控制执行解释的分片。
`source`           | 允许请求的数据放在url的查询字符串中。
`q`                | 查询字符串（映射到`query_string`查询）。
`df`               | 在查询中未定义字段前缀时使用的默认字段。默认为`_all`字段。
`analyzer`         | 分析查询字符串时使用的分析器名称。默认为`_all`字段的分析器。
`analyze_wildcard` | 应该分析通配符和前缀查询。默认为`false`。
`lenient`          | 如果设置为`true`，将会导致基于格式的失败（例如提供一个文本到数字字段）被忽略。默认为`false`。
`default_operator` | 要使用的默认运算符可以是`AND`或`OR`。默认为`OR`。