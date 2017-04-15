# 创建一个索引库

现在让我们创建一个叫做“customer” 的索引，然后再列出所有的索引：

```js
PUT /customer?pretty
GET /_cat/indices?v
```

第一个命令使用PUT创建了一个叫做“customer” 的索引。我们简单地将`pretty`附加到调用的尾部，使其以美观的形式打印出JSON响应。

响应如下：

```js
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```

第二个命令的结果告知我们，我们现在有一个叫做`customer`的索引，并且它有5个主分片和1份副本（都是默认值），其中包含0个文档。

你可能也注意到了这个customer索引有一个黄色健康标签。回顾我们之前的讨论，黄色意味着某些副本没有（或者还未）被分配。这个索引之所以这样，是因为Elasticsearch会默认为这个索引库创建一份副本。 然而由于我们现在只有一个节点在运行，那这份副本就分配不了了（为了高可用），直到另外一个节点加入到这个集群后，才能分配。一旦那份副本被分配到第二个节点，这个索引库的健康状态就会变成绿色。
