# 查询语言介绍

Elasticsearch 提供一种JSON风格的特定领域语言，利用它你可以执行查询。这种语言称为[查询DSL](../../Query_DSL.md)。这个查询语言非常全面，乍一看上去会有点吓人，最好的学习方法就是以几个基础的例子来开始。

回到上一个例子，我们执行了这个查询：

```js
GET /bank/_search
{
  "query": { "match_all": {} }
}
```

分析以上的这个查询，其中的`query`部分告诉我查询的定义，`match_all`部分就是我们想要运行的查询的类型。`match_all`查询，就是简单地查询一个指定索引库下的所有的文档。

除了这个`query`参数之外，我们也可以通过传递其它的参数来影响搜索结果。在之前的示例中我们演示了`sort`，下面演示一下`size`：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```

注意，如果没有指定`size`的值，那么它默认就是10。

下面的例子，做了一次`match_all`查询并且返回第11到第20个文档：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

其中的`from`参数指明从哪个文档开始，`size`参数指明从from参数开始，要返回的文档数。这个特性对于搜索结果分页来说非常有帮助。注意，如果不指定from的值，它默认就是`0`。

下面这个例子演示了`match_all`查询，并且以账户余额降序排序，返前10（默认大小）个文档：

```js
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```