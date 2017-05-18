# Post filter（后置过滤）

在已经计算了聚合之后，`post_filter`应用于搜索请求最后的搜索结果`hits`。其目的最好的例子如下：

想像一下，您正在销售具有以下属性的衬衫：

```js
PUT /shirts
{
    "mappings": {
        "item": {
            "properties": {
                "brand": { "type": "keyword"},
                "color": { "type": "keyword"},
                "model": { "type": "keyword"}
            }
        }
    }
}

PUT /shirts/item/1?refresh
{
    "brand": "gucci",
    "color": "red",
    "model": "slim"
}
```

想象一下，用户已经指定了两个过滤器：

`color:red`与`brand:gucci`。你只想在搜索结果中显示 Gucci 制作的红色衬衫。 通常你会使用[布尔查询](../..//Query_DSL/Compound_queries/Bool_Query.md)：

```js
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  }
}
```

但是，您也可以使用分面导航来显示用户可以点击的其他选项的列表。也许你有一个`model`字段，允许用户将他们的搜索结果限制在红色的Gucci `t-shirts`或者`dress-shirts`。

这可以通过[terms aggregation]()来完成：

```js
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } //①
    }
  }
}
```

① 返回Gucci最受欢迎的红色衬衫款式。

但也许你也想告诉用户Gucci衬衫有多少可用的**其他颜色**。 如果只是在`color`字段上添加`terms`聚合，则只会返回`红色`，因为您的查询只返回Gucci的红色衬衫。

相反，您希望在聚合期间包括所有颜色的衬衫，然后仅将颜色过滤器应用于搜索结果。 这是 `post_filter`的目的：

```js
GET /shirts/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "brand": "gucci" } //①
      }
    }
  },
  "aggs": {
    "colors": {
      "terms": { "field": "color" } //②
    },
    "color_red": {
      "filter": {
        "term": { "color": "red" } //③
      },
      "aggs": {
        "models": {
          "terms": { "field": "model" } //④
        }
      }
    }
  },
  "post_filter": { //⑤
    "term": { "color": "red" }
  }
}
```

&emsp;&emsp;|&emsp;&emsp;
------------|---------------
①          | 主查询现在查找 Gucci 的所有衬衫，而不考虑颜色。
②          | `colors`聚合返回 Gucci 的衬衫的流行颜色。
③,④       | `color_red`聚合将限制在`red` Gucci 衬衫下进行款式子聚合
⑤          | 最后，`post_filter`从搜索`hits`中除去红色以外的颜色。

