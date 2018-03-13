# Field stats API

> 警告
>
> 此功能是实验性功能，可能会在未来版本中完全更改或删除。 Elastic将采取尽最大努力解决任何问题，但实验性功能不受官方GA功能支持SLA的约束。

Field stats api允许在不执行搜索的情况下查找字段的统计属性，但查找Lucene索引中本地可用的度量。这对探索一个你不太了解的数据集很有用。例如，这可以根据最小/最大值范围创建具有有意义间隔的直方图聚合。

 默认情况下，field stats api在所有索引上执行，但也可以在特定索引上执行。
 
 所有索引：

```bash
GET _field_stats?fields=rating
```

指定索引：

```bash
GET twitter/_field_stats?fields=rating
```

支持的请求选项：

名称  | 描述
----|-------
`fields` | 计算统计信息的字段列表。该字段名称支持通配符表示法。例如，使用`text_*`将导致返回与表达式匹配的所有字段。
`level` | 定义是否应在每个索引级别或集群级别上返回字段统计信息。合法值是`indices`和`cluster`（缺省）。

或者，也可以在请求正文中定义fields选项：

```bash
POST _field_stats?level=indices
{
   "fields" : ["rating"]
}
```

这相当于以前的请求。

## 字段统计

field stats api支持基于字符串，基于数字和日期的字段，并且可以返回以下每个字段的统计信息：

字段  |  说明
----|-------
`max_doc` | 文档的总数。
`doc_count` | 至少具有一个字段的文档数量，如果此统计在一个或多个分片上不可用，则为`-1`
`density` |  该字段至少具有一个值的文档的百分比。这是一个派生的统计量，并基于`max_doc`和`doc_count`。
`sum_doc_freq` | 每个术语在此字段中文档频率的总和，如果此测量不适用于一个或多个分片，则为`-1`。文档频率是包含特定术语的文档的数量。
`sum_total_term_freq` | 所有文档中此字段中所有术语的术语频率总和，如果此测量不适用于一个或多个碎片，则为`-1`。术语频率是特定文档和字段中术语出现的总次数。

- `is_searchable`
    - 如果该字段的任何实例都是可搜索的，则为true，否则为false。
- `is_aggregatable`
    - 如果该字段的任何实例都是可聚合的，则为true，否则为false。
`min_value`
    - 该字段中的最低值。
`min_value_as_string`
    - 以可显示的形式表示的字段中的最低值。所有字段，但字符串字段返回此。（因为字符串字段，已将值表示为字符串）
`max_value`
    - 该领域的最高价值。
`max_value_as_string`
    - 以可显示的形式表示的字段中的最高值。所有字段，但字符串字段返回此。（因为字符串字段，已将值表示为字符串）
> 注意
>
> 标记为已删除（但尚未被合并过程删除）的文档仍会影响所有提到的统计信息。

## 集群级别字段统计示例

请求：

```bash
GET _field_stats?fields=rating,answer_count,creation_date,display_name
```

响应：

```bash
{
   "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
   },
   "indices": {
      "_all": {  # ①
         "fields": {
            "creation_date": {
               "max_doc": 1326564,
               "doc_count": 564633,
               "density": 42,
               "sum_doc_freq": 2258532,
               "sum_total_term_freq": -1,
               "min_value": "2008-08-01T16:37:51.513Z",
               "max_value": "2013-06-02T03:23:11.593Z",
               "is_searchable": "true",
               "is_aggregatable": "true"
            },
            "display_name": {
               "max_doc": 1326564,
               "doc_count": 126741,
               "density": 9,
               "sum_doc_freq": 166535,
               "sum_total_term_freq": 166616,
               "min_value": "0",
               "max_value": "정혜선",
               "is_searchable": "true",
               "is_aggregatable": "false"
            },
            "answer_count": {
               "max_doc": 1326564,
               "doc_count": 139885,
               "density": 10,
               "sum_doc_freq": 559540,
               "sum_total_term_freq": -1,
               "min_value": 0,
               "max_value": 160,
               "is_searchable": "true",
               "is_aggregatable": "true"
            },
            "rating": {
               "max_doc": 1326564,
               "doc_count": 437892,
               "density": 33,
               "sum_doc_freq": 1751568,
               "sum_total_term_freq": -1,
               "min_value": -14,
               "max_value": 1277,
               "is_searchable": "true",
               "is_aggregatable": "true"
            }
         }
      }
   }
}
```

① `_al`l键表示它包含集群中所有索引的字段统计信息。

> 注意
>
> 使用集群级别字段统计信息时，如果在具有不兼容类型的不同索引中使用相同字段，则可能会发生冲突。例如，`long`类型的字段与`float`或`string`类型的字段不兼容。如果引发一个或多个冲突，则将名为`conficts`的部分添加到响应中。它包含所有有冲突的领域和不兼容的原因。

```bash
{
   "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
   },
   "indices": {
      "_all": {
         "fields": {
            "creation_date": {
               "max_doc": 1326564,
               "doc_count": 564633,
               "density": 42,
               "sum_doc_freq": 2258532,
               "sum_total_term_freq": -1,
               "min_value": "2008-08-01T16:37:51.513Z",
               "max_value": "2013-06-02T03:23:11.593Z",
               "is_searchable": "true",
               "is_aggregatable": "true"
            }
         }
      }
   },
   "conflicts": {
        "field_name_in_conflict1": "reason1",
        "field_name_in_conflict2": "reason2"
   }
}
```

### 索引库级别的字段统计示例

请求：

```bash
GET _field_stats?fields=rating,answer_count,creation_date,display_name&level=indices
```

响应：

```bash
{
   "_shards": {
      "total": 1,
      "successful": 1,
      "failed": 0
   },
   "indices": {
      "stack": { # ①
         "fields": {
            "creation_date": {
               "max_doc": 1326564,
               "doc_count": 564633,
               "density": 42,
               "sum_doc_freq": 2258532,
               "sum_total_term_freq": -1,
               "min_value": "2008-08-01T16:37:51.513Z",
               "max_value": "2013-06-02T03:23:11.593Z",
               "is_searchable": "true",
               "is_aggregatable": "true"
            },
            "display_name": {
               "max_doc": 1326564,
               "doc_count": 126741,
               "density": 9,
               "sum_doc_freq": 166535,
               "sum_total_term_freq": 166616,
               "min_value": "0",
               "max_value": "정혜선",
               "is_searchable": "true",
               "is_aggregatable": "false"
            },
            "answer_count": {
               "max_doc": 1326564,
               "doc_count": 139885,
               "density": 10,
               "sum_doc_freq": 559540,
               "sum_total_term_freq": -1,
               "min_value": 0,
               "max_value": 160,
               "is_searchable": "true",
               "is_aggregatable": "true"
            },
            "rating": {
               "max_doc": 1326564,
               "doc_count": 437892,
               "density": 33,
               "sum_doc_freq": 1751568,
               "sum_total_term_freq": -1,
               "min_value": -14,
               "max_value": 1277,
               "is_searchable": "true",
               "is_aggregatable": "true"
            }
         }
      }
   }
}
```

① `stack`字段表示`stack`索引库的所有字段统计信息

## 字段统计索引约束

字段统计索引约束允许省略与约束不匹配的索引的所有字段统计信息。索引约束可以根据`min_value`和`max_value`统计量排除索引的字段统计信息。此选项仅在级别选项设置为索引时有用。当索引约束被定义时，未被索引（不可搜索）的字段总是被省略。 

例如，索引约束可用于在基于时间的场景中找出数据的特定属性的最小值和最大值。以下请求仅返回持有2014年创建问题的索引的answer_count属性的字段统计信息：

```bash
POST _field_stats?level=indices
{
   "fields" : ["answer_count"],  # ①
   "index_constraints" : { # ②
      "creation_date" : {  # ③
         "max_value" : {  # ④
            "gte" : "2014-01-01T00:00:00.000Z"
         },
         "min_value" : { # ⑤
            "lt" : "2015-01-01T00:00:00.000Z"
         }
      }
   }
}
```

①   用于计算和返回字段统计信息的字段。
_____________________
②   设置索引约束。请注意，可以为未在字段选项中定义的字段定义索引约束。
_____________________
③   字段creation_date的索引约束。   
_____________________
④,⑤ 字段统计信息的max_value和min_value属性的索引约束。
_____________________

对于字段，可以在`min_value`统计量，`max_value`统计量或两者上定义索引约束。每个索引约束都支持以下比较：

名称   | 描述
-----|------
`gte` | 大于或等于
`gt` | 大于
`lte` | 小于或等于
`lt` | 小于

日期字段上的字段统计索引约束可以接受格式选项，用于分析约束的值。如果缺少，则使用字段映射中配置的格式。

```bash
POST _field_stats?level=indices
{
   "fields" : ["answer_count"],
   "index_constraints" : {
      "creation_date" : {
         "max_value" : {
            "gte" : "2014-01-01",
            "format" : "date_optional_time"  # ①
         },
         "min_value" : {
            "lt" : "2015-01-01",
            "format" : "date_optional_time"
         }
      }
   }
}
```

①  自定义日期格式