# 删除一个文档

现在让我们删除我们刚刚创建的索引库，并再次列出所有的索库：

```js
DELETE /customer?pretty
GET /_cat/indices?v
```

响应如下：

```js
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这表明我们成功地删除了这个索引，现在我们回到了集群刚开始时什么都所有的状态。

再开始之前，我们细看一下我们刚刚学过的API命令：

```js
PUT /customer
PUT /customer/external/1
{
  "name": "John Doe"
}
GET /customer/external/1
DELETE /customer'
```

如果我们仔细研究上面的命令，我们可以看到Elasticsearch是如何访问数据的一个模式。这种模式可以概括如下:

```js
<REST Verb> <Node>:<Port>/<Index>/<Type>/<ID>
```

这个REST访问模式普遍适用于所有的API命令，如果你能记住它，你就会为掌握Elasticsearch开了一个好头。