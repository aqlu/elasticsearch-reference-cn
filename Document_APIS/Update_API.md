# 更新API

更新API允许根据提供的脚本来更新文档。操作从索引获取文档（通过分片并发的），运行脚本（具有可选脚本语言和参数），并将结果建立索引（也允许删除或忽略该操作）。它使用版本控制来确保在“get”和“reindex”期间没有发生更新。

请注意，此操作仍然意味着文档的完整重新索引，它只是删除一些网络往返，并减少`get`和`index`之间版本冲突的机会。需要启用`_source`字段以使此功能正常工作。

为了演示，我们创建一个简单的文档：

```js
PUT test/type1/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

## 脚本更新

现在，我们能执行以下脚本来递增计数器：

```js
POST test/type1/1/_update
{
    "script" : {
        "inline": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
```

我们可以在`tags`中添加一个`tag`（注意，如果`tags`存在，它将会添加它，由于它是一个列表）：

```js
POST test/type1/1/_update
{
    "script" : {
        "inline": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
```

除`_source`外，以下变量可通过`ctx`映射获得：`_index`、`_type`、`_id`、`_version`、`_routing`、`_parent`和`_now`（当前时间戳）。 

我们还可以在文档中添加一个新的字段：

```js
POST test/type1/1/_update
{
    "script" : "ctx._source.new_field = \"value_of_new_field\""
}
```

或者从文档中删除一个字段：

```js
POST test/type1/1/_update
{
    "script" : "ctx._source.remove(\"new_field\")"
}
```

我们甚至可以改变所执行的操作。这个例子中删除文档如果标签字段包含`green`,否则什么也不做(`noop`)：

```js
POST test/type1/1/_update
{
    "script" : {
        "inline": "if (ctx._source.tags.contains(params.tag)) { ctx.op = \"delete\" } else { ctx.op = \"none\" }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
```

## 根据部分文档修改

更新API还支持传递一个部分文档，它将被合并到现有文档中（简单的递归合并，对象的内部合并，替换核心“键/值”和数组）。例如：

```js
POST test/type1/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果指定了`doc`和`script`，则忽略`doc`。最好是将部分文档的字段对放在脚本本身中。

## 探测noop更新

如果指定了`doc`，它的值将与现有的`_source`合并。默认情况下，不更改任何内容的更新会探测到它们不会更改任何内容并返回“result”：“noop”，如下所示：

```js
POST test/type1/1/_update
{
    "doc" : {
        "name" : "new_name"
    }
}
```

如果在发送请求之前`name`是`new_name`，则忽略整个更新请求。如果请求被忽略，则响应中的`result`元素返回`noop`。

```js
{
   "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
   },
   "_index": "test",
   "_type": "type1",
   "_id": "1",
   "_version": 6,
   "result": noop
}
```

您可以通过设置“detect_noop”为`false`来禁用此行为，如下所示：

```js
POST test/type1/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}
```

## Upserts

如果文档不存在，则将`upsert`的内容作为新的文档插入。如果文档确实存在，那么`script`将被执行：

```js
POST test/type1/1/_update
{
    "script" : {
        "inline": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
```

### scripted_upsert

如果您希望无论文档是否存在都运行脚本——即脚本处理初始化文档替代`upsert`元素——则将`scripted_upsert`设置为`true`：

```js
POST sessions/session/dh3sgudg8gsrgl/_update
{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}
```

### doc_as_upsert

将`doc_as_upsert`设置为`true`而不是发送部分文档加上`upsert`文档，而将`doc`的内容用作`upsert`值：

```js
POST test/type1/1/_update
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
```

## 参数

更新操作支持如下查询字符串参数：

参数名                        | 描述
-----------------------------|--------------------
retry_on_conflict            | 在更新的获取和索引阶段之间，另一个进程可能已经更新了同一个文档。默认情况下，更新将失败并出现版本冲突异常。`retry_on_conflict`参数控制在最后抛出异常之前重试更新的次数。
routing                      | 如果正在更新的文档不存在，则使用路由将更新请求路由到正确的分片，并为`upsert`请求设置路由。不能用于更新现有文档的路由。
parent                       | 如果正在更新的文档不存在，则父用于将更新请求路由到正确的分片，并为`upsert`请求设置父级。不能用于更新现有文档的父级。如果指定了别名索引路由，则它将覆盖父路由，并用于路由请求。
timeout                      | 等待分片变为可用的超时时间。
wait_for_active_shards       | 在进行更新操作之前，分片副本的数量需要处于活动状态。详见[这里](./Index_API.md#index-wait-for-active-shards)。
refresh                      | 控制此请求所做的更改对于搜索是可见的。参见[?refresh](./refresh.md)。
_source                      | 允许控制在响应中是否和如何返回更新的源。默认情况下，不会返回更新的源。有关详细信息，请参阅[source filtering](../Search_APIs/Request_Body_Search/Source_filtering.md)。
version & version_type       | 更新API在内部使用Elasticsearch的版本控制支持，以确保在更新过程中文档不会更改。您可以使用`version`参数指定仅当文档的版本与指定的版本匹配时才应更新该文档。通过将版本类型设置为`force`，您可以在更新后强制使用新版本的文档（谨慎使用！`force`不保证文档没有更改）。

> 注意
>
> ## 更新API不支持外部版本控制
> 更新API不支持外部版本控制（版本类型`external`＆`external_gte`），因为它会导致Elasticsearch版本号与外部系统不同步。使用[index API](./Index_API.md)代替。