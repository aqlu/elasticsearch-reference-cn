# Search（搜索）

搜索API允许您执行搜索查询并获取与查询匹配的文档。查询可以使用简单的[查询字符串参数](./URI_Search.md)或[使用请求体](./Request_Body_Search.md)。

## <span id="search-multi-index-type">多索引、多类型</span>

所有搜索API可以跨索引中的多种类型应用，并跨多个索引并支持[多索引语法](../API_Conventions/Multiple_Indices.md)。例如，我们可以搜索`twitter`索引中所有类型的所有文档：

```js
GET /twitter/_search?q=user:kimchy
```

我们也可以指定类型进行搜索：

```js
GET /twitter/tweet,user/_search?q=user:kimchy
```

我们还可以通过多个索引搜索带有某个标签的所有推文（例如，每个用户都有自己的索引）：

```js
GET /kimchy,elasticsearch/tweet/_search?q=tag:wow
```

或者我们可以使用`_all`占位符搜索所有可用索引的所有推文：

```js
GET /_all/tweet/_search?q=tag:wow
```

甚至搜索所有索引库和所有类型：

```js
GET /_search?q=tag:wow
```

默认情况下，`elasticsearch`不会根据请求匹配的分片数拒绝任何搜索请求。虽然elasticsearcg将优化协调节点上的搜索执行，但是大量的分片可以显着影响CPU和内存。组织数据通常是一个更好的主意，这样一来，更小的分片就越少。如果您想要配置软限制，则可以更新`action.search.shard_count.limit`的群集设置，以便拒绝匹配太多分片的搜索请求。
