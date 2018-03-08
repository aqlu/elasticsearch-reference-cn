# Version

返回每个命中文档的版本信息。

```bash
GET /_search
{
    "version": true,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```