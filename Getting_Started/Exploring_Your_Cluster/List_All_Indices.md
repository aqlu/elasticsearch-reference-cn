# 获取所有索引库

让我们看一下我们的索引：

```js
curl '/_cat/indices?v'
```

对应的响应是:

```js
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这个结果意味着，在我们的集群中没有任何索引。