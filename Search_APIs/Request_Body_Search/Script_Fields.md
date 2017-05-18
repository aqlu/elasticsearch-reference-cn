# Script Fields

允许为每次匹配返回[脚本评估](./Modules/Scripting.md)（基于不同的字段），例如：

```js
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "script_fields" : {
        "test1" : {
            "script" : {
                "lang": "painless",
                "inline": "doc['my_field_name'].value * 2"
            }
        },
        "test2" : {
            "script" : {
                "lang": "painless",
                "inline": "doc['my_field_name'].value * factor",
                "params" : {
                    "factor"  : 2.0
                }
            }
        }
    }
}
```

脚本字段可以用于未存储的字段（在上述情况下为`my_field_name`），并允许返回要返回的自定义值（脚本的计算值）。

脚本字段也可以访问实际的_source文档，并通过使用`params['_ source']`提取要返回的特定元素。这是一个例子：

```js
GET /_search
{
    "query" : {
        "match_all": {}
    },
    "script_fields" : {
        "test1" : {
            "script" : "params['_source']['message']"
        }
    }
}
```

注意`_source`关键字在这里用于导航`json`样模型。

了解`doc['my_field'].value`和`params['_source'][my_field]`之间的区别很重要。 第一个，使用`doc`关键字，将导致该字段的词条被加载到内存（缓存），这将导致更快的执行，但更多的内存消耗。 此外，`doc[...]`符号只允许简单的有价值的字段（不能从它返回一个`json`对象），并且只对非分析或单个词条的字段有意义。但是如果可能，使用`doc`仍然是从文档访问值的推荐方法，因为`_source`必须在每次使用时被加载和解析。使用`_source`是非常慢的。