# Rescoring

在使用[query](./Query.md)与[post_filter](./Post_filter.md)语法重新排序返回顶部文档时（例如100-500），使用二次算法（通常跟昂贵），
而不是将昂贵的算法应用于索引中的所有文档。Rescoring能帮助提升精度。

`rescore`请求将在每个分片上执行，然后返回其结果，以便由处理整个搜索请求的节点来进行排序。 

目前`rescore`API只有一个实现：`query rescorer`，它使用查询来调整评分。将来，可以提供备选的`rescorer`，例如，`pair-wise rescorer`。

> 注意：
>
> 在使用[sort](./Sort.md)时，`rescore`语法将不执行。

> 注意：
>
> 当向用户展示分页时，您不应该在逐步浏览每个页面（通过传递不同的值）时更改`window_size`，因为这可能会改变顶部匹配，导致结果在用户逐步浏览页面时发生混乱。

## Query rescorer

`Query rescorer`仅对[query](./Query.md)与[post_filter](./Post_filter.md)阶段返回的`Top-K`结果执行第二次查询。 在每个分片上检查的文档数量可以由`window_size`参数控制，默认为[from和size](./From_Size.md)。

默认情况下，原始查询和`rescore`查询的分数线性组合，以产生每个文档的最终`_score`。 原始查询和`rescore`查询的相对重要性可以分别使用`query_weight`和`rescore_query_weight`进行控制。 两者默认为`1`。

例如：

```bash
curl -s -XPOST 'localhost:9200/_search' -d '{
   "query" : {
      "match" : {
         "field1" : {
            "operator" : "or",
            "query" : "the quick brown",
            "type" : "boolean"
         }
      }
   },
   "rescore" : {
      "window_size" : 50,
      "query" : {
         "rescore_query" : {
            "match" : {
               "field1" : {
                  "query" : "the quick brown",
                  "type" : "phrase",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }
}
'
```

组合得分的方式可以用`score_mode`控制：

Score Mode | Description
----------|----------------------
avg             | 平均原始分数和 rescore 查询分数。 
max            | 取最初的分数和 rescore 查询分数。
min             | 取最初的分数和 rescore 查询分数。
multiply       | 将原始分数乘以 rescore 查询分数。 用于函数查询 rescores。
total           | 添加原始分数和 rescore 查询分数。 默认值。

## Multiple Rescores

也可以按顺序执行多个`rescores`：

```bash
curl -s -XPOST 'localhost:9200/_search' -d '{
   "query" : {
      "match" : {
         "field1" : {
            "operator" : "or",
            "query" : "the quick brown",
            "type" : "boolean"
         }
      }
   },
   "rescore" : [ {
      "window_size" : 100,
      "query" : {
         "rescore_query" : {
            "match" : {
               "field1" : {
                  "query" : "the quick brown",
                  "type" : "phrase",
                  "slop" : 2
               }
            }
         },
         "query_weight" : 0.7,
         "rescore_query_weight" : 1.2
      }
   }, {
      "window_size" : 10,
      "query" : {
         "score_mode": "multiply",
         "rescore_query" : {
            "function_score" : {
               "script_score": {
                  "script": {
                    "lang": "painless",
                    "inline": "Math.log10(doc['numeric'].value + 2)"
                  }
               }
            }
         }
      }
   } ]
}
'
```

第一个获得查询的结果，第二个获得第一个的结果等。第二个`rescore`将 “see” 由第一个`rescore`完成排序，因此可以在第一个`rescore`上使用大窗口 将文档拖动到较小的窗口中，以便第二个文件。