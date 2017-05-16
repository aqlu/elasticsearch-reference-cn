# Bulk API (批量API)

Bulk API可以在单个API调用中执行多个创建索引/删除的操作。这可以大大提高索引速度。

> ## 批量请求客户端支持
>
> 一些官方支持的客户端提供帮助来协助从一个索引到另一个索引的批量请求和重新索引：
>
> Perl
>
> &emsp;&emsp;参见[Search::Elasticsearch::Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk)与[Search::Elasticsearch::Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)
>
> Python
>
> &emsp;&emsp;参见[elasticsearch.helpers.*](http://elasticsearch-py.readthedocs.io/en/master/helpers.html)

REST API端点是`/_bulk`，它期望是以换行符分隔JSON（NDJSON）的结构：

```js
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

注意：最后一行数据必须以换行符`\n`结尾。每个换行符字符之前都可以回车`\r`。当向该端点发送请求时，应将`Content-Type`头设置为`application/x-ndjson`。 

可能的操作是`index`、`create`、`delete`和`update`。`index`和`create`期望下一行的源，并且具有与标准索引API的`op_type`参数相同的语义（即：如果具有相同索引和类型的文档已经存在，则`create`将失败，而有必要时索引将添加或替换文档）。`delete`并不期望下列行的源，并且具有与标准`delete` API相同的语义。 `update`需要在下一行指定部分文档，`upsert`和`script`及其选项。

如果要提供文本文件输入到`curl`，则必须使用`--data-binary`标志替代`-d`的文本。后者不需要保留换行符。例：

```bash
$ cat requests
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
$ curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
{"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"type1","_id":"1","_version":1,"result":"created","forced_refresh":false}}]}
```

因为此格式使用文字`\n`作为分隔符，请确保JSON操作和源文档不是格式化的打印。以下是批量命令正确序列的示例：

```js
POST _bulk
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

批量操作的结果如下：

```js
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "created": true,
            "status": 201
         }
      },
      {
         "delete": {
            "found": false,
            "_index": "test",
            "_type": "type1",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "type1",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "created": true,
            "status": 201
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200
         }
      }
   ]
}
```

端点是`/_bulk`、`/{index}/_bulk`和`{index}/{type}/_bulk`。当提供索引或索引/类型时，它们将作为批量操作的条目的默认值使用、不会作为明确声明的条目使用。

关于格式的注意。这里的想法是尽可能快地处理这个问题。由于某些操作将被重定向到其他节点上的其他分片，因此在接收节点侧仅解析`action_meta_data`。

使用此协议的客户端库应尽可能尝试在客户端执行类似操作，并尽可能减少缓冲。

对批量操作的响应是一个大的JSON结构，其中包含了执行每个操作的各个结果。一个动作的失败不会影响剩余的动作。

单次`bulk`调用没有一个“正确”的操作执行数量。您应该尝试使用不同的设置来查找特定工作负载的最佳大小。

如果使用HTTP API，请确保客户端不发送HTTP块，因为这会减慢事情。

## 版本控制

每个bulk条目可以使用`_version`与`version`字段包含版本值。它基于`_version`映射自动跟踪索引与删除操作的行为。它还支持`version_typ` / `_version_type`（请参阅[版本控制](./Index_API.md#index-versioning)）。

## 路由

每个bulk条目可以使用`_routing`与`routing`字段包括路由值。它基于映射的`_routing`来自动跟踪索引与删除操作的行为。

## Parent

每个bulk条目可以使用`_parent`与`parent`字段来包含父值。它基于映射的`_parent`与`_routing`来自动跟踪索引/删除操作的行为。

## 等待活动分片

进行批量调用时，您可以设置`wait_for_active_shards`参数，以便在开始处理批量请求之前要求最小数量的分片副本处于活动状态。有关详细信息和使用示例，请参阅[此处](./Index_API.md#index-wait-for-active-shards)。


## 冲刷

用来控制本次的修改能够被搜索可见。参见：[refresh](./refresh.md)。

## 更新

当使用`update`操作时，`_retry_on_conflict`可以用作动作本身的字段（而不是额外的数据行），可以指定在版本冲突的情况下应重试更新的次数。

更新操作数据行支持以下选项：`doc`（部分文档）、`upsert`、`doc_as_upsert`、`scirpt`、`params`（与脚本结合使用）、`lang`（与脚本结合使用）和`_source`。有关选项的详细信息，请参阅[更新操作文档](./Update_API.md)。更新操作的示例：

```js
POST _bulk
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "script" : { "inline": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "type1", "_index" : "index1", "_retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "type1", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "type1", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
```

## 安全

参见[基于URL的访问控制](../API_Conventions/URL-based_access_control.md)