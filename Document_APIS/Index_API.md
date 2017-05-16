# Index API（创建索引接口）

`Index API`通过往指定的索引库输入一个JSON文档来添加或更新索引，让文档变得可搜索。以下示例将JSON文档插入到叫 “twitter” 的索引库中，ID 为`1`:

```js
PUT twitter/tweet/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上面的索引操作结果如下：

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 1,
    "created" : true,
    "result" : created
}
```

`_shards` 头提供了有关索引操作的复制过程的信息。

    * `total` —— 标识有多少个分片拷贝被执行创建索引操作（包括主分片与副本分片）。
    * `successful` —— 标识有多少个分片执行创建索引操作成功。
    * `failed` —— 在存在失败的情况下会返回一个操作失败的分片数组列表。

在`successful`至少为`1`的情况下创建索引文档操作才会成功。

> 注意
>
> 当创建索引操作成功时，副本的分片有可能还没开始做（默认情况下，只有主分片才是必须的，但是这个行为是可以[修改](#index-wait-for-active-shards)）。在这个场景中，`total`将等于基于`number_of_replicas`设置的所有分片数，`successful`将会等于已成功的分片数（主分片+副本分片）。如果没有失败的分片，`failed`将会为`0`。


## 自动创建索引库

如果创建索引文档时索引库不存在，这会自动创建索引库（参见手动[创建索引库](../Indices_APIs/Create_Index.md)API），并且会根据动态类型`mapping`来创建索引库的`type`（参见手动[添加映射](../Indices_APIs/Put_Mapping.md)API）。

`mapping`本身是非常灵活与格式自由的。新的字段或者是对象将会被自动添加到指定类型的`mapping`定义中。查看[映射](../Indices_APIs/Get_Mapping.md)部分以获取有关映射定义的更多信息。

可以通过在所有节点的配置文件中将`action.auto_create_index`设置为`false`来禁用自动创建索引库。可以通过将`index.mapper.dynamic`设置为`false`作为索引设置来禁用自动映射创建。

自动创建所以库可以基于一定规则的黑白名单列表，例如，设置`action.auto_create_index` 为 `+aaa*,-bbb*,+ccc*,-*`  （ `+`表示允许，`-`表示不允许）。

## <span id="index-versioning">版本控制</span>

每一个被索引的文档都会给他一个版本号。分配的`version`编号会作为创建索引文档返回结果的一部分。当版本号指定的时候，index API允许[乐观并发控制](http://en.wikipedia.org/wiki/Optimistic_concurrency_control)。这将故意控制文档的版本。一个好的版本控制用户案例是执行事务`读然后修改`。在指定`version`开始读取文档时，同时确保文档不发生变化（读操作在修改操作之后时，推荐设置`perference`为`_primary`）。例如：

```js
PUT twitter/tweet/1?version=2
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

**注意**：版本控制是实时的，不受近实时查询操作的影响。如果没提供版本信息，这个操作的执行将不受到任何版本的检查。

默认情况下，内部版本号从1开始，并且根据每一次的更新、删除操作来递增。可选的是，版本号支持使用外部值（譬如在数据库维护）。要开启此功能，`version_type`需要被设置为`external`。提供的值必须是数字、长整型类型，且大于等于0小于9.2e+18。在使用外部版本控制时，系统检查请求中传递文档的版本号是否大于当前文档的版本号，如果小于等于当前存储的文档版本号会有版本冲突，则会在创建索引文档时返回失败。

> 警告
>
> 外部版本控制支持0是一个有效的版本号。它允许文档的版本在与外部版本控制系统同步时从0开始，而不是1。它的副作用是在版本号为0的文档不能使用[Update-By-Query API](./Update_By_Query_API.md)更新以及不能使用[Delete-By-Query API](./Delete_By_Query_API.md)删除。

一个好的作用是，只要使用源数据库生成的版本号，在执行异步创建文档索引时不需要维护严格顺序。一个简单的场景是，使用数据库数据更新elasticsearch的索引，因为只有最新版本的数据将被使用，不管index操作是否出于什么原因执行乱序了。

### 版本控制类型

上面介绍过了`internal`与`external`，Elasticsearch在特殊的场景中还支持其它的类型。这是不同版本控制类型的概述.

  `internal`

      只能够在指定的版本与存储文档的版本完全一致时创建文档索引。

  `external`或`external_gt`

      如果给定版本严格高于存储的文档的版本或如果没有现有文档，则仅索引文档。给定版本将用作新版本，并与新文档一起存储。提供的版本必须是非负长数字。

  `external_gte`

      如果给定版本高于或等于存储的文档的版本或如果没有现有文档，则仅索引文档。给定版本将用作新版本，并与新文档一起存储。提供的版本必须是非负长数字。

**注意：**`external_gte`版本类型适用于特殊场景，应谨慎使用。如果使用不正确，可能会导致数据丢失。还有一个选项，`force`已被被弃用，因为它可能导致主分片和副本分片出现不一致。

## 操作类型

index操作还可以接受一个`op_type`参数来强制执行`create`操作，允许`put-if-absent`行为。在使用`create`创建文档索引时，如果文档已存在则会失败。

这是使用`op_type`参数的示例：

```js
PUT twitter/tweet/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

还有种通过uri指定`create`的方式

```js
PUT twitter/tweet/1/_create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

## 自动生成ID

可以在不指定 ID 的情况下执行索引操作。在这种情况下，将自动生成`id`。此外，`op_type`将自动设置为`create`。这里是一个例子（注意使用的是**POST**，而不是**PUT**）：

```js
POST twitter/tweet/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

操作将返回：

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "6a8ca01c-7896-48e9-81cc-9f70661fcb32",
    "_version" : 1,
    "created" : true,
    "result": "created"
}
```

## 路由

默认情况下，分片的位置——或路由（routing）——通过使用文档的`id`值的哈希值来控制。对于更明确的控制，基于每个操作直接使用路由参数指定反馈给路由使用的散列函数的值。例如：

```js
POST twitter/tweet?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在上面的示例中，"tweet" 文档根据提供的路由参数发送到分片："kimchy" 。

设置显式映射时，可以选择从文档本身提取路由字段值`_routing`字段来指示索引操作。这在文档解析花费额外的（非常小）成本。如果定义`_routing`映射并将其设置为必需，则如果未提供或提取不到路由值，则索引操作将失败。

## 父文档&子文档

通过在索引时指定其父级，可以索引子文档。例如：

```js
PUT blogs
{
  "mappings": {
    "tag_parent": {},
    "blog_tag": {
      "_parent": {
        "type": "tag_parent"
      }
    }
  }
}

PUT blogs/blog_tag/1122?parent=1111
{
    "tag" : "something"
}
```

对子文档建立索引时，路由值会自动设置为与其父文档相同，除非使用路由参数显式指定路由值。

## 分布式

index操作会根据路由信息（参见上面的路由章节）定向到主分片，并在主分片所在的实际节点上执行。在主分片操作完成后，如果有需要，会分布更新到可用的副本上。

## <span id="index-wait-for-active-shards">等待活动分片</span>

为了提高写入系统的弹性，index操作可以配置为在继续操作之前等待一定数量的活动分片副本。如果所需的活动分片副本数不可用，则写操作必须等待并重试，直到必需的分片副本已启动或发生超时为止。默认情况下，写操作只等待主分片处于活动状态，然后再继续（`wait_for_active_shards=1`）。可以通过设置 `index.write.wait_for_active_shards` 来动态地在索引设置中覆盖此默认值。要更改每个操作的此行为，可以使用 `wait_for_active_shards` 请求参数。

有效值是不大于索引库的所有分片总数的正整数（即`number_of_replicas+1`）。指定负值或者大于分片总数的数字将抛出错误。

例如，假设我们有一个三个节点`A`、`B`和`C`的集群，我们创建一个索引，索引副本数设置为 `3` （会导致有`4`个分片，主分片加3个副本分片）。如果我们尝试索引操作，默认情况下，操作将仅确保每个主分片在继续之前可用。这意味着，在`A`托管主分片时，即使`B`和`C`停机，索引操作仍然将继续，只会有一份数据保存。如果集群有3个活动节点启动，这时每个节点都有一个分片的复制，这时对请求设置 `wait_for_active_shards` 为 `3` （并且所有3个节点都已启动），则索引操作将在继续之前需要 `3` 个活动分片，这是可以执行的。 但如果我们将 `wait_for_active_shards` 设置为 `all` （或者 `4`， 这是相同的），索引操作将不会继续，因为索引中的没有所有的4个分片处于活动状态。该操作将超时，除非在集群中启动新节点以托管分片的第四个副本。

重要的是要注意，这个设置极大地减少了写操作不写入所需数量的分片副本的机会，但是它不能完全消除可能性，因为这种检查在写操作开始之前发生。一旦写操作正在进行，复制仍然可能在任意数量的分片副本上失败，但在主分片上仍然成功。写操作响应的 `_shard` 部分显示复制成功/失败的分片副本的数量。

```js
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

## 冲刷

用来控制本次的修改能够被搜索可见。参见：[refresh](./refresh.md)。

## Noop Updates

当使用index API更新文档时，即使文档没有更改，也始终创建新版本的文档。如果这不可接受，请使用将 `detect_noop` 设置为 `true` 的`_update` API 。此选项在index API上不可用，因为index api不提取旧源，并且无法将其与新源进行比较。

没有一个强制和快速的规则，当等待更新是不能接受的。它是许多因素的组合，例如数据源发送实际上是 noops 的更新的频率，以及每秒多少查询，elasticsearch 在接收更新时在分片上运行。

## 超时

执行索引操作时分配的主分片可能不可用。这种情况的一些原因可能是主分片当前正在从网关恢复或正在进行重分配。默认情况下，索引操作将在主分片上等待最多`1`分钟，然后失败并响应错误。 `timeout`参数可以用于显式指定等待时间。以下是将其设置为`5`分钟的示例：

```js
PUT twitter/tweet/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```