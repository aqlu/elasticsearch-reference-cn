# Get API（提取接口）

get API可以根据跟定的`id`来获取在索引库中存放的文档。下面演示了从一个叫`twitter`的索引库的`tweet` type下获取文档，`id`是`0`:

```js
GET twitter/tweet/0
```

响应结果如下：

```js
{
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上面的结果包含了我们想要提取的文档的`_index`、`_type`、`_id`、`_version`信息，一旦我们找到文档，则真正的文档包含在`_source`下（响应体中的`found`标识了是否找到文档）。

API也可以使用`HEAD`请求来判断文档是否存在，例如：

```js
HEAD twitter/tweet/0
```

## 实时

默认情况下，get API 是实时的，而且它不受创建索引文档的刷新频率的影响（当数据对search操作可见）。如果文档已经修改完，但没还有冲刷，get API将会发起 就地冲刷操作使得文档可见。这也会导致其他文档在上一次冲刷之后发生改变的也会可见。若要禁止`GET`的实时操作，可以设置`realtime`参数为`false`。

## 可选的`Type`

get API允许`_type`是可选的，设置为`_all`后将会从所有的`type`中按顺序返回第一个匹配的文档。

## <span id="get-source-filtering">`Source`过滤</span>

默认情况下，get操作返回`_source`字段的内容，除非你使用`stored_fields`参数或禁用`_source`字段。你可以使用`_source`参数来关闭提取`_source`信息：

```js
GET twitter/tweet/0?_source=false
```

如果你只想从`_source`的全部信息中获取其中的一到两个字段，你可以使用`_source_include`与`_source_exclude`参数来包含或排除你需要的信息部分。这在大型文档中非常有用，部分提取可以节省网络开销。这两个参数都可以使用逗号分隔多个字段以及通配符表达式，例如：

```js
GET twitter/tweet/0?_source_include=*.id&_source_exclude=entities
```

如果你仅需要使用包含某些字段，你可以使用更短的符号：

```js
GET twitter/tweet/0?_source=*.id,retweeted
```

## 存储字段

get 操作允许指定一系列的存储字段，这些字段将会被通过传递`stored_fields`参数返回。如果请求的字段没有被储存，将会被忽略。参考以下示例：

```js
PUT twitter
{
   "mappings": {
      "tweet": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
```

现在我们添加一个文档：

```js
PUT twitter/tweet/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

然后提取它：

```js
GET twitter/tweet/1?stored_fields=tags,counter
```

上面操作的返回结果为：

```js
{
   "_index": "twitter",
   "_type": "tweet",
   "_id": "1",
   "_version": 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档中获取的字段的值通常是返回一个数组。由于`counter`字段没有存储，当尝试获取`stored_fields`时get会将其忽略。

也可以对元数据字段进行检索，比如`_routing`和`_parent`：

```js
PUT twitter/tweet/2?routing=user1
{
    "counter" : 1,
    "tags" : ["white"]
}
```

```js
GET twitter/tweet/2?routing=user1&stored_fields=tags,counter
```

上面操作的返回结果为：

```js
{
   "_index": "twitter",
   "_type": "tweet",
   "_id": "2",
   "_version": 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
}
```

只有叶子字段才可以通过`stored_field`选项返回。所以`object`字段无法返回并且这个请求会失败

## 生成的字段

如果在创建索引和冲刷过程中没有发生冲刷操作，GET会访问事务日志来获取文档。然而，一些字段只会在创建索引时生成，如果你尝试访问那些只有在创建索引时才会创建的字段，默认情况下会得到一个异常。你可以通过设置`ignore_errors_on_generated_fields=true`来忽略那些字段。

## 直接获取`_source`

使用`/{index}/{type}/{id}/_source`可以只获取文档的`_source`字段，不会有其他多余的内容，例如：

```js
GET twitter/tweet/1/_source
```

你也可以使用过滤参数来控制`_source`的哪些部分可以被返回：

```js
GET twitter/tweet/1/_source?_source_include=*.id&_source_exclude=entities'
```

注意，同样可以使用HEAD操作来检测文档的`_source`是否存在。如果它在[mapping](../Mapping/Meta-Fields/_source_field.md)里被禁止，一个存在的文档也不会有`_source`。例如：

```js
HEAD twitter/tweet/1/_source
```

## 路由

当创建索引时想要控制路由，为了获取文档，路由的值也因该提供，例如：

```js
GET twitter/tweet/2?routing=user1
```

以上的操作会获取id为`2`的tweet，但是是基于用户来路由的。注意，如果没有设置正确的路由，将会导致问无法被获取。

## 偏好

可以控制`preference`来让get请求在哪个副本分片上进行搜索，默认情况下，操作将随机在各个副本分片上执行。

`_primary`

    操作将只在主分片上执行。

`_local`

    操作将尽可能的在本地分配的分片上执行。

**自定义（字符串）值**

    自定义值将用于保证相同的分片将使用相同的自定义值。这可以帮助在不同冲刷状态下击中不同分片时的“跳跃值”。譬如可以是一些像会话id或者用户名这样的值。

## 冲刷（Refresh）

`refresh`参数可以被设置为`true`，用来在`get`操作之前冲刷相关的分片，使其变得可搜索。将其设置为`true`应在仔细思考和验证后，不会对系统造成重负荷（并减慢索引）。

## 分布式

get操作被散列到一个指定的分片id。然后被重定向到这个id对于的一个分片并返回结果。这个分片的id是主分片和副本分片组中的一个。这意味着我们在拥有更多的副本时，我们将拥有更好的GET扩展性。

## 版本控制支持

我们可以在提取文档时使用`version`参数，只有在它的当前版本等于指定的版本时才会返回。对于所有版本类型而言，此行为是相同的，但FORCE的版本类型除外，它始终检索文档。注意:`FORCE`版本类型已弃用。

在内部，Elasticsearch已将旧文档标记为已删除，并添加了全新的文档。文档的旧版本不会立即消失，尽管您将无法访问该文档。当您继续索引更多数据时，Elasticsearch会在后台清理已删除的文档。