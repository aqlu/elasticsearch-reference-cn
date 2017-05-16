# 根据查询API更新

`_update_by_query`的最简单的用法只是对索引中的每个文档执行更新，而不会更改源。这对于[收集新的属性](#picking-up-a-new-property)或其他在线映射更改非常有用。这是API：

```js
POST twitter/_update_by_query?conflicts=proceed
```

他将返回类如下的信息：

```js
{
  "took" : 147,
  "timed_out": false,
  "updated": 120,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

`_update_by_query`在启动时获取索引的快照，并使用内部版本控制对其进行索引。这意味着如果文档在拍摄快照和处理索引请求之间发生变化，则会发生版本冲突。当版本匹配文档被更新并且版本号增加。

> 注意
>
> 由于内部版本控制不支持值`0`作为有效的版本号，版本等于零的文档无法使用`_update_by_query`进行更新，并且会使请求失败。

所有更新和查询失败导致`_update_by_query`中止并在响应失败中返回。已执行的更新仍然坚持。换句话说，进程没有回滚，只会中止。当第一个故障导致中止时，失败批量请求返回的所有故障都会返回到故障元素中；因此，有可能会有不少失败的实体。 

如果你想简单地计算版本冲突，不会导致`_update_by_query`中止，你可以在url设置`conflicts=proceed`或在请求体设置`"conflicts": "proceed"`。第一个例子是这样做，因为它只是尝试接管在线映射更改，并且版本冲突只是意味着冲突的文档在`_update_by_query`的开始与尝试更新文档的时间之间更新。这很好，因为该更新将获取在线映射更新。

返回到API格式，您可以将`_update_by_query`限制为单一类型。下面将只从`Twitter`的索引更新`tweet`类型的文件：

```js
POST twitter/tweet/_update_by_query?conflicts=proceed
```

您还可以使用[Query DSL](../Query_DSL.md)限制`_update_by_query`。下面将更新用户`kimchy`的`twitter`索引中的所有文档：

```js
POST twitter/_update_by_query?conflicts=proceed
{
  "query": { //①
    "term": {
      "user": "kimchy"
    }
  }
}
```

① 该查询必须以与[Search API](../Search_APIs/Search.md)相同的方式作为`query`的值传递。您也可以以与搜索api相同的方式使用`q`参数。

到目前为止，我们只是更新文档而不改变它们的来源。这对于[收集新的属性](#picking-up-a-new-property)来说真的很有用，但只有一半的乐趣。 `_update_by_query`支持脚本对象来更新文档。这将增加所有`kimchy`的`tweets`上的`likes`字段：

```js
POST twitter/_update_by_query
{
  "script": {
    "inline": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

就像[更新API](./Update_API.md)一样，您可以设置`ctx.op`来更改执行的操作：

`noop`

  如果您的脚本决定不必进行任何更改，请设置 `ctx.op ="noop"` 。这将导致`_update_by_query` 从其更新中忽略该文档。这个没有操作将被报告在[响应体](#response-body)的 `noop` 计数器上。

`delete`

  如果您的脚本决定必须删除该文档，请设置`ctx.op="delete"`。删除将在[响应体](#response-body)的 `deleted` 计数器中报告。

将`ctx.op`设置为其他任何内容都是错误。在`ctx`中设置任何其他字段是一个错误。

请注意，我们停止指定`conflict=proceed`。在这种情况下，我们希望版本冲突中止该过程，以便我们可以处理该故障。

该API不允许您移动其触摸的文档，只需修改其来源。这是故意的！我们没有规定将文档从原始位置删除。

也可以一次性对多个索引和多个类型进行整体处理，就像搜索API一样：

```js
POST twitter,blog/tweet,post/_update_by_query
```

如果您提供`routing`则将路由复制到滚动查询，将进程限制为与该路由值匹配的分片：

```js
POST twitter/_update_by_query?routing=1
```

默认情况下`_update_by_query`使用滚动批次数量为`1000`.您可以使用URL参数`scroll_size`更改批量大小：

```js
POST twitter/_update_by_query?scroll_size=100
```

`_update_by_query`也可以使用[Ingest Node](../Ingest_Node.md)功能来指定管道，如下所示：

```js
PUT _ingest/pipeline/set-foo
{
  "description" : "sets foo",
  "processors" : [ {
      "set" : {
        "field": "foo",
        "value": "bar"
      }
  } ]
}
POST twitter/_update_by_query?pipeline=set-foo
```

## URL参数

除了标准参数像`pretty`之外，“Update By Query API”还支持`refresh`、`wait_for_completion`、`wait_for_active_shards`、`timeout`以及`requests_per_second`。

发送`refresh`将在更新请求完成时更新索引中的所有分片。这不同于 Index API 的`refresh`参数，只会导致接收到新数据的分片被索引。

如果请求包含`wait_for_completion=false`，那么Elasticsearch将执行一些预检检查、启动请求、然后返回一个任务，可以与[Tasks API](#docs-delete-by-query-task-api)一起使用来取消或获取任务的状态。Elasticsearch还将以`.tasks/task/${taskId}`作为文档创建此任务的记录。这是你可以根据是否合适来保留或删除它。当你完成它时，删除它可以让Elasticsearch回收它使用的空间。

`wait_for_active_shards`控制在继续请求之前必须有多少个分片必须处于活动状态，详见[这里](./Index_API.md#index-wait-for-active-shards)。`timeout`控制每个写入请求等待不可用分片变成可用的时间。两者都能正确地在[Bulk API](./Bulk_API.md)中工作。

`requests_per_second`可以设置为任何正数（1.4，6，1000等），来作为“delete-by-query”每秒请求数的节流阀数字，或者将其设置为`-1`以禁用限制。节流是在批量批次之间等待，以便它可以操纵滚动超时。等待时间是批次完成的时间与`request_per_second * requests_in_the_batch`的时间之间的差异。由于分批处理没有被分解成多个批量请求，所以会导致Elasticsearch创建许多请求，然后等待一段时间再开始下一组。这是“突发”而不是“平滑”。默认值为-1。

## <span id="response-body">响应体</span>

JSON响应类似如下：

```js
{
  "took" : 639,
  "updated": 0,
  "batches": 1,
  "version_conflicts": 2,
  "retries": {
    "bulk": 0,
    "search": 0
  }
  "throttled_millis": 0,
  "failures" : [ ]
}
```

`took`

    从整个操作的开始到结束的毫秒数。

`updated`

    成功更新的文档数。

`batches`

    通过查询更新的滚动响应数量。

`version_conflicts`

    根据查询更新时，版本冲突的数量。

`retries`

    根据查询更新的重试次数。bluk 是重试的批量操作的数量，search 是重试的搜索操作的数量。

`throttled_millis`

    请求休眠的毫秒数，与`requests_per_second`一致。

`failures`

    失败的索引数组。如果这是非空的，那么请求因为这些失败而中止。请参阅 conflicts 来如何防止版本冲突中止操作。

## 配合Task API使用

您可以使用[Task API](../Cluster_APIs/Task_Management_API.md)获取任何正在运行的根据查询修改请求的状态：

```js
GET _tasks?detailed=true&actions=*/update/byquery
```

响应会类似如下：

```js
{
  "nodes" : {
    "r1A2WoRbTwKZ516z6NEs5A" : {
      "name" : "r1A2WoR",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "attributes" : {
        "testattr" : "test",
        "portsfile" : "true"
      },
      "tasks" : {
        "r1A2WoRbTwKZ516z6NEs5A:36619" : {
          "node" : "r1A2WoRbTwKZ516z6NEs5A",
          "id" : 36619,
          "type" : "transport",
          "action" : "indices:data/write/update/byquery",
          "status" : {    //①
            "total" : 6154,
            "updated" : 3500,
            "created" : 0,
            "deleted" : 0,
            "batches" : 4,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": {
              "bulk": 0,
              "search": 0
            }
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}
```

① 此对象包含实际状态。它就像是响应json，重要的添加`total`字段。 `total`是重建索引希望执行的操作总数。您可以通过添加的`updated`、`created`和`deleted`的字段来估计进度。当它们的总和等于`total`字段时，请求将完成。

使用任务id可以直接查找任务：

```js
GET /_tasks/taskId:1
```

这个API的优点是它与`wait_for_completion=false`集成，以透明地返回已完成任务的状态。如果任务完成并且`wait_for_completion=false`被设置，那么它将返回`results`或`error`字段。此功能的成本是`wait_for_completion=false`在`.tasks/task/${taskId}`创建的文档，由你自己删除该文件。

## 配合取消任务API使用

所有根据查询修改都能使用[Task Cancel API](../Cluster_APIs/Task_Management_API.md)取消：

```js
POST _tasks/task_id:1/_cancel
```

可以使用上面的任务API找到`task_id`。

取消应尽快发生，但可能需要几秒钟。上面的任务状态API将继续列出任务，直到它被唤醒取消自身。

## 重置节流阀

`request_per_second`的值可以在通过查询删除时使用`_rethrottle` API更改：

```js
POST _update_by_query/task_id:1/_rethrottle?requests_per_second=-1
```

可以使用上面的任务API找到task_id。 

就像在`_update_by_query` API中设置它一样，`request_per_second`可以是`-1`来禁用限制，或者任何十进制数字，如1.7或12，以节制到该级别。加速查询的会立即生效，但是在完成当前批处理之后，减慢查询的才会生效。这样可以防止滚动超时。

## 手动切片

根据查询修改支持[滚动切片](../Search_APIs/Request_Body_Search/Scroll.md#sliced-scroll)，您可以相对轻松地手动并行化处理：

```js
POST twitter/_update_by_query
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "script": {
    "inline": "ctx._source['extra'] = 'test'"
  }
}
POST twitter/_update_by_query
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "script": {
    "inline": "ctx._source['extra'] = 'test'"
  }
}
```

您可以通过以下方式验证：

```js
GET _refresh
POST twitter/_search?size=0&q=extra:test&filter_path=hits.total
```

其结果一个合理的`total`像这样：

```js
{
  "hits": {
    "total": 120
  }
}
```

## 自动切片

你还可以让根据查询修改使用切片的`_uid`来自动并行的[滚动切片](../Search_APIs/Request_Body_Search/Scroll.md#sliced-scroll)。

```js
POST twitter/_update_by_query?refresh&slices=5
{
  "script": {
    "inline": "ctx._source['extra'] = 'test'"
  }
}
```

您可以通过以下方式验证：

```js
POST twitter/_search?size=0&q=extra:test&filter_path=hits.total
```

其结果一个合理的`total`像这样：

```js
{
  "hits": {
    "total": 120
  }
}
```

将`slices`添加到`_update_by_query`中可以自动执行上述部分中使用的手动过程，创建子请求，这意味着它有一些怪癖：

- 您可以在[Task API](#docs-delete-by-query-task-api)中看到这些请求。这些子请求是具有`slices`请求任务的“子”任务。
- 获取`slices`请求任务的状态只包含已完成切片的状态。
- 这些子请求可以单独寻址，例如取消和重置节流阀。
- `slices`的重置节流阀请求将按相应的重新计算未完成的子请求。
- `slices`的取消请求将取消每个子请求。
- 由于`slices`的性质，每个子请求将不会获得完全均匀的文档部分。所有文件都将被处理，但有些片可能比其他片大。预期更大的切片可以有更均匀的分布。
- 带有`slices`请求的`request_per_second`和`size`的参数相应的分配给每个子请求。结合上述关于分布的不均匀性，您应该得出结论，使用切片大小可能不会导致正确的大小文档为`_update_by_query`。
- 每个子请求都会获得源索引的略有不同的快照，尽管这些都是大致相同的时间。

## 挑选切片数量

在这一点上，我们围绕要使用的`slices`数量提供了一些建议（比如手动并行化时，切片API中的`max`参数）：

- 不要使用大的数字，`500`就能造成相当大的CPU抖动。
- 从查询性能的角度来看，在源索引中使用分片数量的一些倍数更为有效。
- 在源索引中使用完全相同的分片是从查询性能的角度来看效率最高的。
- 索引性能应在可用资源之间以`slices`数量线性扩展。
- 索引或查询性能是否支配该流程取决于许多因素，如正在重建索引的文档和进行`reindexing`的集群。

## <span id="picking-up-a-new-property">收集新的属性</span>

假设您创建了一个没有动态映射的索引，用数据填充它，然后添加一个映射值来从数据中获取更多的字段：

```js
PUT test
{
  "mappings": {
    "test": {
      "dynamic": false,   //①
      "properties": {
        "text": {"type": "text"}
      }
    }
  }
}

POST test/test?refresh
{
  "text": "words words",
  "flag": "bar"
}
POST test/test?refresh
{
  "text": "words words",
  "flag": "foo"
}
PUT test/_mapping/test   //②
{
  "properties": {
    "text": {"type": "text"},
    "flag": {"type": "text", "analyzer": "keyword"}
  }
}
```

① 这意味着新的字段将不会被索引，只存储在_source中。
____________________________________________
② 将更新映射以添加新的`flag`字段。要接收新的字段，你必须重新索引所有的文档。

搜索数据将找不到任何内容：

```js
POST test/_search?filter_path=hits.total
{
  "query": {
    "match": {
      "flag": "foo"
    }
  }
}
```

```js
{
  "hits" : {
    "total" : 0
  }
}
```

但是您可以发出`_update_by_query`请求来接收新映射：

```js
POST test/_update_by_query?refresh&conflicts=proceed
POST test/_search?filter_path=hits.total
{
  "query": {
    "match": {
      "flag": "foo"
    }
  }
}
```

```js
{
  "hits" : {
    "total" : 1
  }
}
```

将字段添加到多字段时，您可以执行完全相同的操作。