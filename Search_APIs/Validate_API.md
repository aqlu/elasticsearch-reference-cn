# Validate API

validate API 允许用户验证一个可能复杂（expensive）的查询而不执行它。 我们将使用以下测试数据来解释`_validate`：

```bash
PUT twitter/tweet/_bulk?refresh
{"index":{"_id":1}}
{"user" : "kimchy", "post_date" : "2009-11-15T14:12:12", "message" : "trying out Elasticsearch"}
{"index":{"_id":2}}
{"user" : "kimchi", "post_date" : "2009-11-15T14:12:13", "message" : "My username is similar to @kimchy!"}
```

当发送一个有效查询时：

```bash
GET twitter/_validate/query?q=user:foo
```

响应包含有效：`true`

```bash
{"valid":true,"_shards":{"total":1,"successful":1,"failed":0}}
```

## Request Parameters

当执行查询使用查询参数q时，传递的查询是使用Lucene查询解析器的查询字符串。 还有其他可以传递的参数：

名称   | 描述
-----|------
'df' | 在查询中未定义字段前缀时使用的默认字段。
'analyzer' | 分析查询字符串时使用的分析器名称。
'default_operator' | 要使用的默认运算符，可以是`AND`或`OR`。 默认为`OR`。
'lenient' | 如果设置为`true`将导致基于格式的失败（例如向数字字段提供文本）被忽略。 默认为`false`。
'lowercase_expanded_terms' | 术语是否自动小写，默认为`true`。
'analyze_wildcard' | 是否分析通配符和前缀查询。 默认为`false`。


查询也可以在请求主体中发送：

```bash
GET twitter/tweet/_validate/query
{
  "query" : {
    "bool" : {
      "must" : {
        "query_string" : {
          "query" : "*:*"
        }
      },
      "filter" : {
        "term" : { "user" : "kimchy" }
      }
    }
  }
}
```

> 注意
>
> 在正文中发送的查询必须嵌套在查询键中，与[Search API](../Search_APIs/Search.md)相同。

如果查询无效，则返回信息中`valid`将为`false`。 在这里，查询无效，因为 Elasticsearch知道`post_date`字段应该是动态映射的日期，`foo`无法正确解析为日期：

```bash
GET twitter/tweet/_validate/query?q=post_date:foo
```
 
 ```js
{"valid":false,"_shards":{"total":1,"successful":1,"failed":0}}
```

可以指定`explain`参数以获取有关查询失败原因的更详细信息：

```bash
GET twitter/tweet/_validate/query?q=post_date:foo&explain=true
```

响应是：

```js
{
  "valid" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "explanations" : [ {
    "index" : "twitter",
    "valid" : false,
    "error" : "twitter/IAEc2nIXSSunQA_suI0MLw] QueryShardException[failed to create query:...failed to parse date field [foo]"
  } ]
}
```

当查询有效时，`explanations`默认为该查询的字符串表示形式。 将 `rewrite`设置为`true`时，`explanations`将更详细地显示将要执行的实际Lucene查询。

## 模糊查询（Fuzzy Queries）：

```bash
GET twitter/tweet/_validate/query?rewrite=true
{
  "query": {
    "match": {
      "user": {
        "query": "kimchy",
        "fuzziness": "auto"
      }
    }
  }
}
```

响应：

```js
{
   "valid": true,
   "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
   },
   "explanations": [
      {
         "index": "twitter",
         "valid": true,
         "explanation": "+user:kimchy +user:kimchi^0.75 #(ConstantScore(_type:tweet))^0.0"
      }
   ]
}
```

## 相似度查询（More Like This）：

```bash
GET twitter/tweet/_validate/query?rewrite=true
{
  "query": {
    "more_like_this": {
      "like": {
        "_id": "2"
      },
      "boost_terms": 1
    }
  }
}
```

响应：

```js
{
   "valid": true,
   "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
   },
   "explanations": [
      {
         "index": "twitter",
         "valid": true,
         "explanation": "((user:terminator^3.71334 plot:future^2.763601 plot:human^2.8415773 plot:sarah^3.4193945 plot:kyle^3.8244398 plot:cyborg^3.9177752 plot:connor^4.040236 plot:reese^4.7133346 ... )~6) -ConstantScore(_uid:tweet#2)) #(ConstantScore(_type:tweet))^0.0"
      }
   ]
}
```

> 警告
>
> 请求只在单个分片上执行，这是随机选择的。 查询的详细解释可以取决于哪个分片被命中，并且因此可以从一个请求到另一个请求而变化。