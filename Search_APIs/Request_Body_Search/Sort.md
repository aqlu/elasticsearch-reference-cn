# Sort

允许在特定字段上添加一个或多个排序。 每个排序也可以颠倒。 排序是在每个字段级别上定义的，具有`_score`的特殊字段名称按分数排序，`_doc`按索引顺序排序。

假设以下索引映射：

```js
PUT /my_index
{
    "mappings": {
        "my_type": {
            "properties": {
                "post_date": { "type": "date" },
                "user": {
                    "type": "keyword"
                },
                "name": {
                    "type": "keyword"
                },
                "age": { "type": "integer" }
            }
        }
    }
}
```

```js
GET /my_index/my_type/_search
{
    "sort" : [
        { "post_date" : {"order" : "asc"}},
        "user",
        { "name" : "desc" },
        { "age" : "desc" },
        "_score"
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

> 注意：
>
> `_doc`除了能最有效率的排列顺序、没有真正的使用场景。所以如果你不关心文档返回的顺序，那么你应该按`_doc`排序。 这特别有助于[滚动](./Scroll.md)。

## 排序值

返回的每个文档的排序值也作为响应的一部分返回。

## 排列顺序

排序选项可以有以下值：

选项   | 描述
------|--------------
`asc` | 升序
`desc`| 降序

在对`_score`进行排序时，该顺序默认为`desc`，在对其他事物进行排序时默认为`asc`。

## 排列模式选项

Elasticsearch支持按数组或多值字段排序。`mode`选项控制选择用于对其所属文档进行排序的数组值。 `mode`选项可以具有以下值：

选项     | 描述
--------|--------------
`min`   | 选择最低值。
`max`   | 选择最高值。
`sum`   | 使用所有值的和作为排序值。 仅适用于基于数字的数组字段。
`avg`   | 使用所有值的平均值作为排序值。 仅适用于基于数字的数组字段。
`median`| 使用所有值的中值作为排序值。 仅适用于基于数字的数组字段。

### 排序模式用法示例

在下面的示例中，每个文档字段价格有多个价格。 在这种情况下，结果匹配将按基于每个文档的平均价格的升序排序。

```js
PUT /my_index/my_type/1?refresh
{
   "product": "chocolate",
   "price": [20, 4]
}

POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
      {"price" : {"order" : "asc", "mode" : "avg"}}
   ]
}
```

## 使用内嵌对象排序

Elasticsearch 还支持根据一个或多个嵌套对象内的字段进行排序。 通过嵌套字段支持进行的排序在已经存在的排序选项之上具有以下参数：

`nested_path`

    定义要排序的嵌套对象。 实际排序字段必须是此嵌套对象内的直接字段。 当通过嵌套字段排序时，此字段是必需的。

`nested_filter`

    嵌套路径中的内部对象应与其匹配的过滤器，以便通过排序考虑其字段值。 常见的情况是在嵌套的过滤器或查询中重复查询/过滤。 默认情况下，没有 nested_filter 是激活的。

### 内嵌对象排序示例

在下面的示例中，`offer`是一个类型为嵌套的字段。 需要指定`nested_path`; 否则，elasticsearch不知道需要捕获哪个嵌套级排序值。

```js
POST /_search
{
   "query" : {
      "term" : { "product" : "chocolate" }
   },
   "sort" : [
       {
          "offer.price" : {
             "mode" :  "avg",
             "order" : "asc",
             "nested_path" : "offer",
             "nested_filter" : {
                "term" : { "offer.color" : "blue" }
             }
          }
       }
    ]
}
```

内嵌对象排序同样也支持脚本排序和按地理距离排序。

## 缺失值

`missing`参数指定如何处理缺少此字段的文档：`missing`的值可以设置为`_last`，`_first`或自定义值（将用于缺失字段的文档的排序值）。

例如：

```js
GET /_search
{
    "sort" : [
        { "price" : {"missing" : "_last"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```

> 注意
>
> 如果嵌套的内部对象与`nested_filter`不匹配，则使用缺少的值。

## 忽略没有映射的字段

默认地，如果没有与字段关联的映射，搜索请求将失败。 `unmapped_type`选项允许忽略没有映射且没有由它们排序的字段。 此参数的值用于确定要发出的排序值。下面是一个如何使用它的例子：

```js
GET /_search
{
    "sort" : [
        { "price" : {"unmapped_type" : "long"} }
    ],
    "query" : {
        "term" : { "product" : "chocolate" }
    }
}
```

如果查询的任何索引没有`price`的映射，那么Elasticsearch将处理它，就好像存在类型为`long`的映射，其中该索引中的所有文档都没有该字段的值。

## GEO距离排序

允许按`_geo_distance`排序。 下面是一个例子，假设`pin.location`是一个类型为`geo_point`的字段：

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km",
                "mode" : "min",
                "distance_type" : "arc"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

`distance_type`

    如何计算距离。 如何计算距离。可以是弧（默认）或平面（更快，但长距离不准确，靠近极点）。

`mode`

    如果字段有多个地理点，该怎么办。 默认情况下，按升序排序时考虑最短距离，按降序排序时最长距离。 支持的值为 min，max，median 和 avg。

`unit`

    计算排序值时使用的单位。 默认值为 m（米）。

> 注意
>
> `geo distance sorting`不支持可配置的缺失值：当文档没有用于距离计算的字段的值时，距离将始终被视为等于`Infinity`。

在提供坐标时支持以下格式：

### 属性格式的经纬度

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : {
                    "lat" : 40,
                    "lon" : -70
                },
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 字符串格式的经纬度

在`lat`，`lon`中的格式。

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : "40,-70",
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### GeoHash

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : "drm3btev3e86",
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

### 经纬度数组

格式是`[lon,lat]`，注意，`lon/lat`的顺序在这里为了符合[GeoJSON](http://geojson.org/)。

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [-70, 40],
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 多个引用点

多个地理点可以作为一个包含任何 geo_point 格式的数组传递，例如，

```js
GET /_search
{
    "sort" : [
        {
            "_geo_distance" : {
                "pin.location" : [[-70, 40], [-71, 42]],
                "order" : "asc",
                "unit" : "km"
            }
        }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

等等。

文档的最终距离将是包含在文档中的所有点的最小/最大/平均（通过模式定义）到在排序请求中给出的所有点的距离。

## 基于脚本排序

允许基于自定义脚本排序，这里是一个例子：

```js
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    },
    "sort" : {
        "_script" : {
            "type" : "number",
            "script" : {
                "lang": "painless",
                "inline": "doc['field_name'].value * params.factor",
                "params" : {
                    "factor" : 1.1
                }
            },
            "order" : "asc"
        }
    }
}
```

## 追踪分数

在字段上排序时，不会计算分数。 通过将`track_scores`设置为`true`，仍将计算和跟踪分数。

```js
GET /_search
{
    "track_scores": true,
    "sort" : [
        { "post_date" : {"order" : "desc"} },
        { "name" : "desc" },
        { "age" : "desc" }
    ],
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

## 内存注意事项

排序时，相关的排序字段值将加载到内存中。这意味着每个分片应该有足够的内存来容纳它们。对于基于字符串的类型，排序的字段不应该被`analyzed`或`tokenized`。对于数字类型，如果可能，建议将类型显式设置为较窄类型（如`short`，`integer`和`float`）。