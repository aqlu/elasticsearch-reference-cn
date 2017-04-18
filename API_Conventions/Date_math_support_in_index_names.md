# 索引库名称的日期运算

通过日期计算索引名称可以解决查询一个时间范围内的索引库，而不是通过别名来搜索所有连续时间的索引库来过滤得到结果。限制搜索的索引数量，减少了集群上的负载，并提高了执行性能。例如，如果您在日常日志中搜索错误信息），则可以使用日期计算名称模板将搜索限制为过去两天。

几乎所有的具有`index`参数的API，都支持通过日期计算`index`的参数值。

日期计算索引名称采用以下形式：

```js
<static_name{date_math_expr{date_format|time_zone}}>
```

说明：

上式位置         | 含义说明
----------------|--------------------------
static_name     | 是名称的静态文本部分
date_math_expr  | 是一个日期计算动态表达式，用来动态计算日期
date_format     | 是计算日期应呈现的格式参数，可选。默认是`YYYY.MM.dd`
time_zone       | 是可选的，时区。默认为`utc`。

你必须将索引名称与日期计算表达式包含在尖括号中，并且所有的特殊字符都应进行URI编码。例如：

```js
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

> 注意
>
> ## 日期运算符的URI编码
>
> 用于日期运算的特殊字符必须按照如下URI编码：
>
> 特殊字符  | 百分比编码
> ---------|---------
> <        | %3C
> \>       | %3E
> /        | %2F
> {        | %7B
> }        | %7D
> \|       | %7C
> \+       | %2B
> :        | %3A

以下示例显示不同形式日期运算后的索引名称，它们在解析时给出的当前时间是`2024年3月22日中午`的utc时间。

表达式                            | 解析结果
---------------------------------|--------------
<logstash-{now/d}>               |logstash-2024.03.22
<logstash-{now/M}>               |logstash-2024.03.01
<logstash-{now/M{YYYY.MM}}>      |logstash-2024.03
<logstash-{now/M-1M{YYYY.MM}}>   |logstash-2024.02
<logstash-{now/d{YYYY.MM.dd|+12:00}}>|logstash-2024.03.23

如要在索引名称的静态部分中使用字符`{`和`}` ，使用`\`对其进行转义，例如：

* `<elastic\\{ON\\}-{now/M}>`将被解析为`elastic{ON}-2024.03.01`

下面的示例显示了一个搜索请求，在过去的三天里搜索LogStash索引，假设索引使用默认的索引名称格式， `logstash-YYYY.MM.dd`。

```js
# GET /<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd-2d%7D%3E%2C%3Clogstash-%7Bnow%2Fd-1d%7D%3E%2C%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```