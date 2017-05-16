# Multi Get API

Multi GET API允许基于索引、类型（可选）和id（以及可能的路由）获取多个文档。响应包括一个具有所有获取的文档的`docs`数组，每个元素的结构与[get](./Get_API.md) API提供的文档相似。这是一个例子：

```js
GET _mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
```

`mget`端点也可以用于索引（在这种情况下，它不需要在主体中）：

```js
GET test/_mget
{
    "docs" : [
        {
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
```

以及类型：

```js
GET test/type/_mget
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
```

在这种情况下，`ids`元素可以直接用于简化请求：

```js
GET test/type/_mget
{
    "ids" : ["1", "2"]
}
```

## 可选的类型

mget API允许`_type`是可选的。将其设置为`_all`或将其留空，以便获取与所有类型的`id`匹配的第一个文档。 

如果您没有设置类型，并且有很多文档共享相同的`_id`，您将最终只得到第一个匹配的文档。

例如，如果您在类型A和类型B中有文档`1`，则以下请求将仅返回相同的文档两次：

```js
GET test/_mget
{
    "ids" : ["1", "1"]
}
```

在这种情况下需要明确设置`_type`：

```js
GET test/_mget/
{
  "docs" : [
        {
            "_type":"typeA",
            "_id" : "1"
        },
        {
            "_type":"typeB",
            "_id" : "1"
        }
    ]
}
```

## Source过滤

默认情况下，将为每个文档（如果存储）返回`_source`字段。与[get](./Get_API.md) API类似，您只能使用`_source`参数来检索`_source`（或不是所有）的部分。您还可以使用url参数`_source`、`_source_include`和`_source_exclude`来指定默认值，当没有每个文档的指令时，它将被使用。

 例如：

 ```js
 GET _mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
 ```

## 字段

可以根据Get API的[stored_fields](./Get_API.md#get-stored-fields)参数指定特定的存储字段，以便每个文档检索。例如：

```js
GET _mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
```

或者，您可以将查询字符串中的`stored_fields`参数指定为默认值以应用于所有文档。

```js
GET test/type/_mget?stored_fields=field1,field2
{
    "docs" : [
        {
            "_id" : "1" //①
        },
        {
            "_id" : "2",
            "stored_fields" : ["field3", "field4"] //②
        }
    ]
}
```

① 返回`field1`与`field2`
________________________
② 返回`field3`与`field4`

## 生成的字段

有关仅在创建索引时生成的字段，请参见[生成的字段章节](./Get_API.md#generated-fields)。


## 路由

你可以通过参数值指定路由值：

```js
GET _mget?routing=key1
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
```

在这个例子中，文档`test/type/2`将从对应于路由键`key1`的分片中获取，但是文档`test/type/1`将从对应于路由键`key2`的分片中获取。

## 安全

参见[基于URL的访问控制](../API_Conventions/URL-based_access_control.md)