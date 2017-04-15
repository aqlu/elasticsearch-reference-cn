# 更新文档

除了可以索引、替换文档之外，我们也可以更新一个文档。但要注意，Elasticsearch底层并不支持原地更新。在我们想要做一次更新的时候，Elasticsearch先删除旧文档，然后再索引新的文档。

下面的例子展示了怎样将ID为1的文档的name字段改成“Jane Doe”：

```js
POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe" }
}
```

下面的例子展示了怎样将ID为1的文档的name字段改成“Jane Doe”的同时，给它加上age字段：

```js
POST /customer/external/1/_update?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```

更新也可以通过使用简单的脚本来进行。这个例子使用一个脚本将age加5：

```js
POST /customer/external/1/_update?pretty
{
  "script" : "ctx._source.age += 5"
}
```

在上面的例子中，`ctx._source`指向当前被更新的文档。

注意，目前的更新操作只能一次修改在一个文档上。将来Elasticsearch将提供同时更新符合指定查询条件的多个文档的功能（类似于SQL的`UPDATE-WHERE`语句）。
