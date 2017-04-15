# 执行搜索

现在我们已经知道了几个基本的参数， 让我们进一步学习查询语言。首先我们看一下返回文档的字段。 默认情况下，是返回完整的JSON文档的。这可以通过`source`来引用（搜索`hits`中的`_source`字段）。如果我们不想返回完整的源文档，我们可以指定返回的几个字段。

下面这个例子说明了从搜索中只返回两个字段`account_number`和`balance`（这两个字段都必须包含在`_source`中），如下：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

注意到上面的例子简化了`_source`字段,它仍将会返回一个叫做`_source`的字段，但是只会包含`account_number`和`balance`两个子字段。

如果你有SQL经验，上述查询在概念上有些像SQL的 `SELECT 字段1, 字段2 FROM TABLE_1`。

现在让我们进入到查询部分。之前，我们学习了`match_all`查询是怎样匹配到所有的文档的。现在我们介绍一种新的查询，叫做`match`查询，这可以看成是一个简单的字段搜索（比如对某个或某些特定字段的搜索）

下面这个例子返回账户编号为20的文档：

```js
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

下面这个例子返回地址中包含了“mill”词条(term)的所有账户：

```js
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```

下面这个例子返回地址中包含“mill”或者“lane”词条的账户：

```js
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

下面这个例子是`match`的变体（`match_phrase`），它会去匹配短语“mill lane”：

```js
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

现在，让我们介绍一下[布尔查询](../..//Query_DSL/Compound_queries/Bool_Query.md)。布尔查询允许我们利用布尔逻辑将较小的查询组合成较大的查询。

现在这个例子组合了两个`match`查询，这个组合查询返回同时包含了“mill” 和“lane” 的所有的账户：

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的例子中，`bool` `must`语句指明了对于一个文档，所有的查询都必须为真，这个文档才能够匹配成功。

相反的， 下面的例子组合了两个`match`查询，它返回的是地址中可能包含了“mill” 或者“lane”的所有的账户:

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的例子中`bool` `should`语句指明，对于一个文档，查询列表中，只要有一个查询匹配，那么这个文档就被看成是匹配的。

现在这个例子组合了两个查询，它返回地址中既不包含“mill”，同时也不包含“lane”的所有的账户信息：

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的例子中，`bool` `must_not`语句指明，对于一个文档，查询列表中的的所有查询都必须都不为真，这个文档才被认为是匹配的。

我们可以在一个bool查询里一起使用`must`、`should`、`must_not`。 此外，我们可以将bool查询放到这样的bool语句中来模拟复杂的、多层级的布尔逻辑。

下面这个例子返回40岁以上并且不生活在ID（aho）的人的账户：

```js
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```