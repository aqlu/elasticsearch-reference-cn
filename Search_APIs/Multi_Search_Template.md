# Multi Search Template

多搜索（Multi Search）模板 API 允许使用`_msearch/template`端点在同一个`API`中执行多个搜索模板请求。

以下请求形式与[Multi Search API](./Multi_Search_API.md)形式相似：

```js
header\n
body\n
header\n
body\n
```

`header`部分与一般的Multi Search API支持相同的`index`，`types`，`search_type`，`preference`和`routing`选项。

`body`部分包含一个搜索模板`body`请求，并且支持`inline`，存储和文件模板。比如：

```bash
$ cat requests
{"index": "test"}
{"inline": {"query": {"match":  {"user" : "{{username}}" }}}, "params": {"username": "john"}} # ①
{"index": "_all", "types": "accounts"}
{"inline": {"query": {"{{query_type}}": {"name": "{{name}}" }}}, "params": {"query_type": "match_phrase_prefix", "name": "Smith"}}
{"index": "_all"}
{"id": "template_1", "params": {"query_string": "search for these words" }} # ②
{"types": "users"}
{"file": "template_2", "params": {"field_name": "fullname", "field_value": "john smith" }} # ③
 
$ curl -XGET localhost:9200/_msearch/template --data-binary "@requests"; echo
```

① `inline`搜索模板请求
________________________
② 基于存储模板的搜索模板请求
________________________
③ 基于文件模板的搜索模板请求
________________________

响应返回一个`responses`数组，其中包括每个搜索模板请求的搜索模板响应，该响应匹配其在原始多搜索模板请求中的顺序。如果该特定搜索模板请求的完全失败，将返回`error`消息的对象，而不是实际的搜索响应。