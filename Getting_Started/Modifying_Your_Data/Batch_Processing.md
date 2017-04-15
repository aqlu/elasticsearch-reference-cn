# 批量处理

除了能够对单个的文档进行索引、更新和删除之外，Elasticsearch也提供了操作的批量处理功能，它通过使用[_bulk API](../../Document_APIS/Bulk_API.md)实现。这个功能非常重要，因为它提供了非常高效的机制来尽可能快的完成多个操作，与此同时尽可能地减少网络交互。

作为一个快速的例子，以下调用在一次bulk操作中索引了两个文档（ID 1 - John Doe 与 ID 2 - Jane Doe） :

```js
POST /customer/external/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

以下例子在一个bulk操作中，首先更新第一个文档（ID为1），然后删除第二个文档（ID为2）

```js
POST /customer/external/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

注意上面的delete动作，由于删除动作只需要被删除文档的ID，所以并没有对应的源文档。

Bulk API不会因为其中的一个动作失败而整体失败。如果其中一个动作因为某些原因失败了，它将会继续处理后面的动作。在Bulk API返回时，它将提供每个动作的状态（按照同样的顺序），所以你能够看到某个动作成功与否。