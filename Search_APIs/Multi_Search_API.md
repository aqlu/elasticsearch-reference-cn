# Multi Search API

Multi search API允许在同一个API中执行多个搜索请求。它的端点是`_msearch`。 

请求的格式类似于批量API格式，并使用换行符分隔的`JSON（NDJSON）`格式。结构如下（如果特定搜索结束重定向到另一个节点，则结构被特别优化以减少分析）：

```bash
header\n
body\n
header\n
body\n
```
**注意**：最后一行数据必须以换行符`\n`结尾。每个换行符都可以在回车添加`\r`。向这个端点发送请求时，`Content-Type`头应该被设置为`application / x-ndjson`。

header包括要搜索的 *index/indices* ，可选项有*types*，`search_type`， `preference` 和 `routing`。 正文包括典型的搜索正文请求（包括`query`, `aggregations`, `from`, `size` 等）。 这里是一个例子：

```bash
$ cat requests
{"index" : "test"}
{"query" : {"match_all" : {}}, "from" : 0, "size" : 10}
{"index" : "test", "search_type" : "dfs_query_then_fetch"}
{"query" : {"match_all" : {}}}
{}
{"query" : {"match_all" : {}}}
 
{"query" : {"match_all" : {}}}
{"search_type" : "dfs_query_then_fetch"}
{"query" : {"match_all" : {}}}
```

```bash 
$ curl -H "Content-Type: application/x-ndjson" -XGET localhost:9200/_msearch --data-binary "@requests"; echo
```

注意，上面包括也被支持的空标题（也可以只是没有任何内容）的示例。

该响应返回一个响应数组，其中包括每个搜索请求的搜索响应和状态代码，与其在原始 multi search 请求中的顺序相匹配。 如果该特定搜索请求的完全失败，将返回具有错误消息和相应状态代码的对象，而不是实际的搜索响应。

端点还允许对URI中的 index/indices 和 type/types 进行搜索，在这种情况下，它将被用作默认值，除非在标题中另有明确定义。 例如：

```bash 
GET twitter/_msearch
{}
{"query" : {"match_all" : {}}, "from" : 0, "size" : 10}
{}
{"query" : {"match_all" : {}}}
{"index" : "twitter2"}
{"query" : {"match_all" : {}}}
```

以上将针对所有未定义索引的请求执行针对`twitter`索引的搜索，最后一个针对`twitter2`索引执行。

` search_type`可以以类似的方式设置为全局应用于所有搜索请求。 

msearch的`max_concurrent_searches`请求参数可用于控制多搜索api将执行的最大并发搜索数。此默认值基于数据节点的数量和默认搜索线程池大小。

## 安全

请查看[URL-based access control](../API_Conventions/URL-based_access_control.md)


## 模板支持

就像`_search`资源的[搜索模板](./Search_Template.md)中所述，`_msearch`也提供对模板的支持。提交他们如下所示：

```bash
GET _msearch/template
{"index" : "twitter"}
{ "inline" : "{ \"query\": { \"match\": { \"message\" : \"{{keywords}}\" } } } }", "params": { "query_type": "match", "keywords": "some message" } }
{"index" : "twitter"}
{ "inline" : "{ \"query\": { \"match_{{template}}\": {} } }", "params": { "template": "all" } }
```

为内联模板。 您也可以创建搜索模板：

```bash
POST /_search/template/my_template_1
{
    "template": {
        "query": {
            "match": {
                "message": "{{query_string}}"
            }
        }
    }
}
```

```bash
POST /_search/template/my_template_2
{
    "template": {
        "query": {
            "term": {
                "{{field}}": "{{value}}"
            }
        }
    }
}
```

然后在_msearch使用它们:

```bash
GET _msearch/template
{"index" : "main"}
{ "id": "my_template_1", "params": { "query_string": "some message" } }
{"index" : "main"}
{ "id": "my_template_2", "params": { "field": "user", "value": "test" } }
```