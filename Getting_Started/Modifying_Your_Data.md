# 修改数据

Elasticsearch提供了近乎实时的数据操作和搜索功能。默认情况下，从你索引/更新/删除你的数据动作开始到它出现在你的搜索结果中，大概会有1秒钟的延迟（刷新间隔）。这和其它的SQL平台不同，它们的数据在一个事务完成之后就会立即可用。

## 索引/替换文档

我们先前看到，怎样索引一个文档。现在我们再次调用那个命令：

```js
PUT /customer/external/1?pretty
{
  "name": "John Doe"
}
```

以上的命令将会把这个文档索引到customer索引的external类型中，其ID是1。如果我们对一个不同（或相同）的文档应用以上的命令，Elasticsearch将会用一个新的文档来替换（重新索引）当前ID为1的那个文档。

```js
PUT /customer/external/1?pretty
{
  "name": "Jane Doe"
}
```

以上的命令将ID为1的文档的name字段的值从“John Doe” 改成了“Jane Doe”。如果我们使用一个不同的ID，一个新的文档将会被索引，当前已经在索引中的文档则不会受到影响。

```js
PUT /customer/external/2?pretty
{
  "name": "Jane Doe"
}
```

以上的命令，将会创建一个ID为2的索引文档。

在索引的时候，ID部分是可选的。如果不指定，Elasticsearch将产生一个随机的ID来索引这个文档。Elasticsearch 生成的ID会作为索引API调用的一部分被返回。

下面的例子展示了怎样在没有指定ID的情况下来索引一个文档：

```js
POST /customer/external?pretty
{
  "name": "Jane Doe"
}
```

注意，在上面的情形中，由于我们没有指定一个ID，我们使用的是`POST`而不是`PUT`。

