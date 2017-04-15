# 探索您的数据

## 样本数据集

现在我们对于基本的东西已经有了一些认识，现在让我们尝试使用一些更加贴近真实的数据集。我准备了一些假想的银行账户信息的JSON文档样本。文档具有以下的模式（schema）：

```js
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

不要好奇，我是从[www.json-generator.com/](http://www.json-generator.com/)自动生成这些数据的，所以请忽略这些数据的真实意义。

## 载入样本数据

你可以在[这里](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)下载样本数据集。将其解压到当前目录下并加载到我们的集群里：

```js
curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
```

响应是：

```js
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb
```

这意味着我们成功批量索引了1000个文档到`bank`索引中（在account类型下）。