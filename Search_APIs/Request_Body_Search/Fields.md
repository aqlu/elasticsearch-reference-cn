# Fields

> 警告
>
> `stored_fields`参数是关于显式标记为存储在映射中的字段，默认情况下关闭，通常不推荐。 使用[源过滤](./Source_filtering.md)来选择要返回的原始源文档的子集。

允许有选择地加载搜索匹配所表示的每个文档的特定存储字段。

```js
GET /_search
{
    "stored_fields" : ["user", "postDate"],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`*`可用于从文档加载所有存储的字段。

空数组只会为每个匹配返回`_id`和`_type`，例如：

```js
GET /_search
{
    "stored_fields" : [],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

如果请求的字段未存储（`store`映射设置为`false`），它们将被忽略。

从文档本身获取的存储字段值总是作为数组返回。 相反，诸如`_routing`和`_parent`字段的元数据字段从不作为数组返回。

此外，只有叶子字段（leaf field）可以通过字段选项返回。 因此，无法返回对象字段，并且此类请求将失败。

脚本字段也可以自动检测并用作字段，所以像`_source.obj1.field1`这样的东西可以使用，虽然不推荐，因为`obj1.field1`也会工作。

## 完全禁用存储字段

要禁用存储的字段（和元数据字段），请完全使用`_none_`：

```js
GET /_search
{
    "stored_fields": "_none_",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

> 注意
>
> 如果使用`_none_` , 则[_source](./Source_filtering.md)和[version](./Version.md)参数将不可用。
