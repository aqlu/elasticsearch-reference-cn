# Source filtering

允许控制`_source`字段如何在每次的采样中返回。

默认操作返回`_source`字段的内容，除非您已使用`stored_fields`参数或禁用`_source`字段。

您可以使用`_source`参数关闭对`_source`的提取：

要禁用`_source`的提取可以设置其为`false`：

```js
GET /_search
{
    "_source": false,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`_source`还接受一个或多个通配符模式来控制`_source`的哪些部分应该返回：

例如：

```js
GET /_search
{
    "_source": "obj.*",
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

或

```js
GET /_search
{
    "_source": [ "obj1.*", "obj2.*" ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

最后，为了完全控制，您可以指定包含和排除模式：

```js
GET /_search
{
    "_source": {
        "includes": [ "obj1.*", "obj2.*" ],
        "excludes": [ "*.description" ]
    },
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```