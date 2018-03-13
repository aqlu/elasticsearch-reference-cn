# Suggesters

`suggest`特性通过使用`suggester`基于所提供的文本来建议相似的术语。部分`suggest`功能还在开发中。

suggest请求部分是在查询的`_search`请求中定义的。

> 注意：
>
> `_suggest`端点已被弃用，倾向于通过`_search`端点使用suggest。在5.0中，`_search`端点针对`suggest`的搜索请求进行过优化。

```bash
POST twitter/_search
{
  "query" : {
    "match": {
      "message": "tring out Elasticsearch"
    }
  },
  "suggest" : {
    "my-suggestion" : {
      "text" : "trying out Elasticsearch",
      "term" : {
        "field" : "message"
      }
    }
  }
}
```

可以为每个请求指定几条建议。每个建议都以任意名称标识。在下面的例子中，需要两个建议。`my-suggest-1`与`my-suggest-2`都使用`term`建议器，但有不同的`text`。

```bash
POST _search
{
  "suggest": {
    "my-suggest-1" : {
      "text" : "tring out Elasticsearch",
      "term" : {
        "field" : "message"
      }
    },
    "my-suggest-2" : {
      "text" : "kmichy",
      "term" : {
        "field" : "user"
      }
    }
  }
}
```

下面的 suggest 响应示例包括对于`my-suggest-1`和 `my-suggestion-2`的 suggest 响应。 每个suggest 部分包含条目（entries）。 每个条目实际上是来自 suggest 文本的 token ，并且包含 suggest 文本中的 suggest 条目文本，原始的条目开始于 suggest 偏移（offset）和长度，并且如果找到任意数目的选项。

```js
{
  "_shards": ...
  "my-suggest-1": [ {
    "text": "tring",
    "offset": 0,
    "length": 5,
    "options": [ {"text": "trying", "score": 0.8, "freq": 1 } ]
  }, {
    "text": "out",
    "offset": 6,
    "length": 3,
    "options": []
  }, {
    "text": "elasticsearch",
    "offset": 10,
    "length": 13,
    "options": []
  } ],
  "my-suggest-2": ...
}
```

每个选项数组（option array）包含一个选项对象，其中包括 suggest 文本，其文档频率和分数与 suggest 输入文本相比较。 分数的意义取决于使用的suggester。 术语 suggester 的分数是基于编辑（edit）距离。

## 全局 suggest 文本

为了避免重复 suggest 文本，可以定义全局文本。 在下面的示例中，suggest 文本是全局定义的，并适用于`my-suggest-1`和`my-suggest-2`建议。

```bash
POST _suggest
{
  "text" : "tring out Elasticsearch",
  "my-suggest-1" : {
    "term" : {
      "field" : "message"
    }
  },
  "my-suggest-2" : {
    "term" : {
      "field" : "user"
    }
  }
}
```

在上述示例中，suggest 文本也可以被指定为 suggest 特定选项。 在 suggestion 级别上指定的 suggest 文本覆盖全局级别上的 suggest 文本。

 