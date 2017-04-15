# 执行聚合

聚合提供了分组并统计数据的能力。理解聚合的最简单的方式是将其粗略地等同为SQL的GROUP BY和SQL聚合函数。在Elasticsearch中，你可以在一个响应中同时返回命中的数据和聚合结果。你可以使用简单的API同时运行查询和多个聚合并一次返回，这避免了来回的网络通信，是非常强大和高效的。

首先，这个例子演示了根据账户所在的州来统计各个州的账户数，并返回账户数最多的前10个州（默认大小为10）：

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

在SQL中，上面的聚合在概念上类似于：

```sql
SELECT COUNT(*) from bank GROUP BY state ORDER BY COUNT(*) DESC
```

响应（其中一部分）是：

```js
 {
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

我们可以看到`ID`（Idaho）州有27个账户，紧跟着`TX`（Texas）州有27个账户、`AL`（Alabama）有25个账户，依此类推。

注意我们将`size`设置成 0，这样我们就可以只看到聚合结果了，而不会显示命中的文档的详细结果。

在先前聚合的基础上，现在这个例子计算了每个州的账户的平均存款（还是按照账户数量倒序排序的前10个州）：

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

注意， 我们把`average_balance`聚合嵌套在了`group_by_state`聚合之中。这是所有聚合的一个常用模式。你可以在任意的聚合之中嵌套聚合，这样就可以从你的数据中抽取出想要的结果。

在前面的聚合的基础上，现在让我们按照平均余额进行排序：

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

下面的例子显示了如何使用年龄段（20-29，30-39，40-49）分组，然后再用性别分组，最后为每一个年龄段的每组性别计算平均账户余额。

```js
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```
