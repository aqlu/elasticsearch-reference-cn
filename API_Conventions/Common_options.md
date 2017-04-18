# 常用选项

以下选项可以应用于所有 REST APIs 。

## 优雅的结果

当对任何请求追加`?pretty=true`时，返回的JSON将会优雅地格式化（仅用于调试！）。另一个选项是设置`?format = yaml`，这将使结果以（有时）更可读的 `yaml`格式返回。

## 人类可读的输出（Human readable output）

以适合人类的格式（例如 "exists_time":"1h" 或者 "size":"1kb"）和计算机（例如 "exists_time_in_millis":"3600000" 或者 "size_in_bytes":"1024"）返回统计信息。通过向查询字符串中添加`?human=false`可以关闭人类可读的值。当统计结果被监视工具消费而不是用于人类消费时，这将是有意义的。人性化标志的默认值是`false`。

## 日期运算

接受格式化的日期值的参数大多数都支持日期运算——譬如使用`gt`与`lt`的[范围查询](../Query_DSL/Term_level_queries/Range_Query.md)，或者使用`from`与`to`的[日期范围聚合](../Aggregations/Bucket_Aggregations/Date_Range_Aggregation.md)。

表达式以锚定日期开始，可以是`now`，也可以`||`结尾的日期字符串。此锚定日期可以选择性地后跟一个或多个数学表达式：

数学表达式 | 含义
----------|-------
+1h       | 加一个小时（add one hour）
-1d       | 减去一天（subtract one day）
/d        | 向下舍入到最近的一天（round down to the nearest day）

所支持的时间单位不同于[持续时间支持的时间单位](#time-units)。所支持的单位是：

符号 | 含义
-----|----
y    | years
M    | months
w    | weeks
d    | days
h    | hours
H    | hours
m    | minutes
s    | seconds

下面是一些例子：

表达式              | 含义
-------------------|--------------------
now+1h             | 当前时间加上一个小时，以毫秒（ms）为单位
now+1h+1m          | 当前时间加上一个小时一分钟，以毫秒（ms）为单位
now+1h/d           | 当前时间加上一个小时，向下舍入到最近的一天。
2015-01-01||+1M/d  | 2015-01-01 加一个月，向下舍入到最近一天。

## 响应过滤（respoonse filtering）

所有REST API都支持可用于减少elasticsearch返回的`filter_path`参数。此参数是采用`,`分隔，且支持`.`符号表达：

```js
GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score
```

响应（Responds）：

```js
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "0",
        "_score" : 1.6375021
      }
    ]
  }
}
```

并且`*`通配符可以用于包括不知道确切路径的字段。例如：

```js
GET /_cluster/state?filter_path=routing_table.indices.**.state
```

响应（Responds）：

```js
{
  "routing_table": {
    "indices": {
      "twitter": {
        "shards": {
          "0": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "1": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "2": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "3": [{"state": "STARTED"}, {"state": "UNASSIGNED"}],
          "4": [{"state": "STARTED"}, {"state": "UNASSIGNED"}]
        }
      }
    }
  }
}
```

也可以通过使用字符`-`过滤器来排除一个或多个字段：

```js
GET /_count?filter_path=-_shards
```

响应（Responds）：

```js
{
  "count" : 5
}
```

为了更多的控制，包含和独占过滤器可以组合在同一个表达式。在这种情况下，将首先应用独占过滤器，并使用包含过滤器再次过滤：

```js
GET /_cluster/state?filter_path=metadata.indices.*.state,-metadata.indices.logstash-*
```

响应（Responds）：

```js
{
  "metadata" : {
    "indices" : {
      "index-1" : {"state" : "open"},
      "index-2" : {"state" : "open"},
      "index-3" : {"state" : "open"}
    }
  }
}
```

请注意， `elasticsearch`有时直接返回字段的原始值，如`_source`字段。如果要过滤`_source`字段，你需要结合`filter_path`参数来考虑已存在`_source`参数。（请参阅[Get API](../Document_APIS/Get_API.md#get-source-filtering)了解更多详细信息 ），如下所示：

```js
POST /library/book?refresh
{"title": "Book #1", "rating": 200.1}
POST /library/book?refresh
{"title": "Book #2", "rating": 1.7}
POST /library/book?refresh
{"title": "Book #3", "rating": 0.1}
GET /_search?filter_path=hits.hits._source&_source=title&sort=rating:desc
```

```js
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
```

## 扁平设置

`flat_settings`参数影响设置列表的呈现。当`flat_settings`设置为`true`时，结果将以扁平的格式返回：

```js
GET twitter/_settings?flat_settings=true
```

返回（Returns）：

```js
{
  "twitter" : {
    "settings": {
      "index.number_of_replicas": "1",
      "index.number_of_shards": "1",
      "index.creation_date": "1474389951325",
      "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
      "index.version.created": ...,
      "index.provided_name" : "twitter"
    }
  }
}
```

当`flat_settings`设置为`false`时，设置以更易于阅读的结构化格式返回：

```js
GET twitter/_settings?flat_settings=false
```

返回（Returns）：

```js
{
  "twitter" : {
    "settings" : {
      "index" : {
        "number_of_replicas": "1",
        "number_of_shards": "1",
        "creation_date": "1474389951325",
        "uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "version": {
          "created": ...
        },
        "provided_name" : "twitter"
      }
    }
  }
}
```

默认情况下， `flat_settings`被设置为`false`。

## 参数（Parameters）

Rest 参数（当使用 HTTP 时，映射到 HTTP URL 参数）遵循使用下划线框的惯例。

## 布尔值（Boolean Values）

所有REST APIs参数（请求参数和 JSON 正文）支持以下值作为布尔值“false”：`false`，`0`，`no`和`off`。所有其他值均被视为“true”。

> 警告
>
> ### 5.3.0已废弃
>
> 除“false”与“true”之外的其他值都被废弃。

## 数值

所有REST API都支持以字符串的形式来提供数字，以支持JSON数字类型。

## <span id="time-units">时间单位（Time units）</span>

每当需要指定持续时间时，譬如`timeout`参数，持续时间必须指定单位，如`2d`为`2天`。支持的单位有：

符号   | 含义
------|------
d     | days
h     | hours
m     | minutes
s     | seconds
ms    | milliseconds
micros| microseconds
nanos | nanoseconds

## 字节大小单位

每当需要指定数据的字节大小时，例如，当设置缓冲区（buffer）大小参数时，该值必须指定单位，例如`10`千字节的`10kb`。支持的单位有：

单位|全称
---|---
b  | Bytes
kb | Kilobytes
mb | Megabytes
gb | Gigabytes
tb | Terabytes
pb | Petabytes


## 无单位数量（unit-less quantities）

无单位数量意味着它们没有像“字节（bytes）”或者“赫兹（Hertz）”或者“米（meter）”或者“长吨（long tonne）” 的“单位”。
如果这些数量中的一个很大，我们将打印出来，如10,000万的 10m 或者 7,000 的 7k 。我们仍然打印 87 ，当我们的意思是 87 。这些是支持的乘数（multipliers）：

符号  |含义
------|-------
\`\` | Single
k    | Kilo
m    | Mega
g    | Giga
t    | Tera
p    | Peta

## 距离单位（distance units）

无论在何处需要指定距离，例如“地理距离查询”中的距离参数，默认单位（如果没有指定）是 米（meter）。距离可以用其他单位指定，例如 “1公里（km）”或者“2公里（mi）”（2英里）。

单位的完整列表如下：

单位         |表示符号
-------------|----------
Mile         | mi 或者 miles
Yard         | yd 或者 yards
Feet         | ft 或者 feet
Inch         | in 或者 inch
Kilometer    | km 或者 kilometers
Meter        | m 或者 meters
Centimeter   | cm 或者 centimeters
Millimeter   | mm 或者 millimeters
Nautical mile| NM, nmi 或者 nauticalmiles

## 模糊性（Fuzziness）

一些查询和 APIs 支持参数以允许使用模糊性参数进行不精确的模糊匹配。

当查询文本（text）或者关键字字段（keyword fields）时，模糊性被解释为 Levenshtein Edit Distance —— 需要对一个字符串进行更改以使其与另一个字符串相同的一个字符的数目。

`fuzziness`参数可以指定为：

  `0， 1， 2`

    最大允许 Levenshtein Edit Distance （或者编辑次数）。

  `AUTO`

    基于该项的长度生成编辑距离（generates an edit distance）。对于长度：

      0..2

         必须完全匹配

      3..5

        允许编辑一次（one edit allowed）

      >5

        允许编辑两次（two edits allowed）

AUTO一般应该是`fuzziness`的首选值。

## 启用堆栈跟踪

默认情况下，当请求返回错误时，Elasticsearch不包括错误的堆栈跟踪。您可以将`error_trace`参数设置为`true`来启用该行为。例如，默认情况下，当您向`_search`API 发送无效的`size`参数时：

```js
POST /twitter/_search?size=surprise_me
```

响应看起来像下面这样：

```js
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Failed to parse int parameter [size] with value [surprise_me]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "Failed to parse int parameter [size] with value [surprise_me]",
    "caused_by" : {
      "type" : "number_format_exception",
      "reason" : "For input string: \"surprise_me\""
    }
  },
  "status" : 400
}
```

但是，如果您设置`error_trace=true`：

```js
POST /twitter/_search?size=surprise_me&error_trace=true
```

响应看起来像这样：

```js
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Failed to parse int parameter [size] with value [surprise_me]",
        "stack_trace": "Failed to parse int parameter [size] with value [surprise_me]]; nested: IllegalArgumentException..."
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Failed to parse int parameter [size] with value [surprise_me]",
    "stack_trace": "java.lang.IllegalArgumentException: Failed to parse int parameter [size] with value [surprise_me]\n    at org.elasticsearch.rest.RestRequest.paramAsInt(RestRequest.java:175)...",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"surprise_me\"",
      "stack_trace": "java.lang.NumberFormatException: For input string: \"surprise_me\"\n    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)..."
    }
  },
  "status": 400
}
```

## 请求体作为查询字符串

对于不接受请求体的`non-POST`请求，你可以把请求体粘贴到查询字符串的`source`参数来执行。在使用吃方法时，需要通过`source_content_type`参数指定媒体类型来表明数据来源的格式，譬如：`application/json`。

## Content-Type自动探测

参数内容通过请求主体或者是查询字符串`source`来传递时会自动探测内容的类型（JSON、YAML、SMILE、或者CBOR）。

严格的模式可以开启禁用自动探测，然后需要在请求头的`Content-Type`中设置支持的格式。若要开启严格模式，请添加如下行到`elasticsearch.yml`文件：

```yaml
http.content_type.required: true
```

它的默认值为false。