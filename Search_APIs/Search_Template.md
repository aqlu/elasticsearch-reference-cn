# Search Template

`/_search/template`端点允许使用mustache语言在执行前预渲染搜索请求，并使用模板参数填充现有模板。

```bash
GET /_search/template
{
    "inline" : {
      "query": { "match" : { "{{my_field}}" : "{{my_value}}" } },
      "size" : "{{my_size}}"
    },
    "params" : {
        "my_field" : "foo",
        "my_value" : "bar",
        "my_size" : 5
    }
}
```

关于`Mustache templating`以及你可以使用哪种模板，请参考 [`mustache`项目在线文档](http://mustache.github.io/mustache.5.html)。

> 注意：
>
> `mustache`语言在elasticsearch中作为沙盒脚本语言实现，因此对于启用或禁用，源代码和操作等设置，它遵从[脚本文档](../Modules/Scripting/Scripting_and_security.md#script_source_settings)的描述。

## 更多的模板案例

### 使用单个值填充查询字符串

```bash
GET /_search/template
{
    "inline": {
        "query": {
            "match": {
                "title": "{{query_string}}"
            }
        }
    },
    "params": {
        "query_string": "search for these words"
    }
}
```

### 将参数转换为 JSON

`{{#toJson}}parameter{{/toJson}}`函数可以用来转换参数（比如`maps`和`array`）为它们的`JSON`形式。

```bash
GET /_search/template
{
  "inline": "{ \"query\": { \"terms\": { \"status\": {{#toJson}}status{{/toJson}} }}}",
  "params": {
    "status": [ "pending", "published" ]
  }
}
```

其呈现为：

```js
{
  "query": {
    "terms": {
      "status": [
        "pending",
        "published"
      ]
    }
  }
}
```

更复杂的例子代替一个 JSON 对象数组：

```bash
{
    "inline": "{\"query\":{\"bool\":{\"must\": {{#toJson}}clauses{{/toJson}} }}}",
    "params": {
        "clauses": [
            { "term": "foo" },
            { "term": "bar" }
        ]
   }
}
```

呈现为：

```js
{
    "query" : {
      "bool" : {
        "must" : [
          {
            "term" : "foo"
          },
          {
            "term" : "bar"
          }
        ]
      }
    }
}
```

### 连接数组的值

`{{#join}}array{{/join}}`函数能用来将数组的值连接为以逗号分隔的字符串：

```bash
GET /_search/template
{
  "inline": {
    "query": {
      "match": {
        "emails": "{{#join}}emails{{/join}}"
      }
    }
  },
  "params": {
    "emails": [ "username@email.com", "lastname@email.com" ]
  }
}
```

其呈现为：

```js
{
    "query" : {
        "match" : {
            "emails" : "username@email.com,lastname@email.com"
        }
    }
}
```

该函数还可以接受一个自定义分隔符：

```bash
GET /_search/template
{
  "inline": {
    "query": {
      "range": {
        "born": {
            "gte"   : "{{date.min}}",
            "lte"   : "{{date.max}}",
            "format": "{{#join delimiter='||'}}date.formats{{/join delimiter='||'}}"
            }
      }
    }
  },
  "params": {
    "date": {
        "min": "2016",
        "max": "31/12/2017",
        "formats": ["dd/MM/yyyy", "yyyy"]
    }
  }
}
```

呈现为：

```js
{
    "query" : {
      "range" : {
        "born" : {
          "gte" : "2016",
          "lte" : "31/12/2017",
          "format" : "dd/MM/yyyy||yyyy"
        }
      }
    }
}
```

### 默认值

默认值被写成`{{var}}{{^var}}default{{/var}}`比如：

```js
{
  "inline": {
    "query": {
      "range": {
        "line_no": {
          "gte": "{{start}}",
          "lte": "{{end}}{{^end}}20{{/end}}"
        }
      }
    }
  },
  "params": { ... }
}
```

此时`params`是 `{ "start": 10, "end": 15 }`，该查询可以呈现为：

```js
{
    "range": {
        "line_no": {
            "gte": "10",
            "lte": "15"
        }
  }
}
```

但是，` params`是`{ "start": 10 }`时， 该查询就会使用默认值作为`end`:

```js
{
    "range": {
        "line_no": {
            "gte": "10",
            "lte": "20"
        }
    }
}
```

### 条件语句

条件语句不能使用模板的 JSON 形式表示。相反，模板必须作为字符串传递。例如，假设我们想在`line`字段上运行`match`查询，并且可以选择用行号过滤其中`start`和`end`是可选的。

参数（params）会像这样：

```bash
{
    "params": {
        "text": "words to search for",
        "line_no": {    # ①
            "start": 10, # ②
            "end": 20   # ③
        }
    }
}
```

①、②、③   所有这三个元素都是可选的。
______________________________

查询语句我们可以是：

```bash
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "line": "{{text}}" # ①
        }
      },
      "filter": {
        {{#line_no}}  # ②
          "range": {
            "line_no": {
              {{#start}}  # ③
                "gte": "{{start}}" # ④
                {{#end}},{{/end}} # ⑤
              {{/start}} # ⑥
              {{#end}} # ⑦
                "lte": "{{end}}" # ⑧
              {{/end}} # ⑨
            }
          }
        {{/line_no}} # ⑩
      }
    }
  }
}
```

① 填充`text`参数值。
______________________________
② 仅当指定`line_no`时才能包含`range`过滤器。
______________________________
③ 仅当指定了`line_no.start`时才能包含`gte`子句。
______________________________
④ 填充`line_no.start`参数值。
______________________________
⑤ 仅当`line_no.start`和`line_no.end`被指定时，在`gte`子句后添加逗号。
______________________________
⑦⑨ 仅当指定`line_no.end`时，包含`lte`子句。
______________________________
⑩ 填充 line_no.end 参数值。
______________________________

> 注意：
>
> 如上所述，模板不是有效的JSON 形式，因为它包含类似`{{line_no}}`的部分标记，因此，模板应存储在文件中（参考[预注册模板](#pre-registered-templates)一节）或者在通过`REST API`使用时，应该将其写成字符串格式，比如：

```bash
{
  "inline": "{\"query\":{\"bool\":{\"must\":{\"match\":{\"line\":\"{{text}}\"}},\"filter\":{{{#line_no}}\"range\":{\"line_no\":{{{#start}}\"gte\":\"{{start}}\"{{#end}},{{/end}}{{/start}}{{#end}}\"lte\":\"{{end}}\"{{/end}}}}{{/line_no}}}}}}"
}
```

### URL编码

可以使用`{{#url}}value{{/ url}}`函数按[HTML规范](http://www.w3.org/TR/html4/)中定义的HTML编码格式对字符串值进行编码。 

作为一个例子，编码一个URL是很有用的：

```bash
GET _render/template
{
    "inline" : {
        "query" : {
            "term": {
                "http_access_log": "{{#url}}{{host}}/{{page}}{{/url}}"
            }
        }
    },
    "params": {
        "host": "https://www.elastic.co/",
        "page": "learn"
    }
}
```

前面的查询将呈现为：

```js
{
    "template_output" : {
        "query" : {
            "term" : {
                "http_access_log" : "https%3A%2F%2Fwww.elastic.co%2F%2Flearn"
            }
        }
    }
}
```

 

### <span id="pre-registered-templates">预注册（Pre-registered）模板</span>

你可以通过将搜索模板存储在`config/scripts`目录中，在使用 `.mustache`扩展名的文件中注册搜索模板。为了执行存储的模板，请使用`template`键下的名称来引用它：

```bash
GET /_search/template
{
    "file": "storedTemplate",  # ①
    "params": {
        "query_string": "search for these words"
    }
}
```

① `config/sripts/`查询模板的名称，即：`storedTemplate.mustache`。
______________________________

你还可以通过将搜索模板存储在集群状态中来注册搜索模板，由`REST API`来管理这些索引的模板。

```bash
POST /_search/template/<templatename>
{
    "template": {
        "query": {
            "match": {
                "title": "{{query_string}}"
            }
        }
    }
}
```

该模板可以通过以下方法来检查：

```bash
GET /_search/template/<templatename>
```

其可以呈现为：

```js
{
    "template": {
        "query": {
            "match": {
                "title": "{{query_string}}"
            }
        }
    }
}
```

这个模板可以被删除：

```bash
DELETE /_search/template/<templatename>
```

在搜索时使用索引模板，使用：

```bash
GET /_search/template
{
    "id": "templateName",  # ①
    "params": {
        "query_string": "search for these words"
    }
}
```

① 存储在 .scripts 索引中的查询模板的名称。
______________________________

### 验证（Validating）模板

可以在具有给定参数的响应中使用来呈现模板：

```bash
GET /_render/template
{
  "inline": {
    "query": {
      "terms": {
        "status": [
          "{{#status}}",
          "{{.}}",
          "{{/status}}"
        ]
      }
    }
  },
  "params": {
    "status": [ "pending", "published" ]
  }
}
```

此次调用将返回呈现的模板：

```js
{
  "template_output": {
    "query": {
      "terms": {
        "status": [  # ①
          "pending",
          "published"
        ]
      }
    }
  }
}
```

① status 数据已使用来自 params 对象的值填充。
______________________________
 
文件和索引模板也可以通过分别用`file`或`id`代替`inline`来呈现。例如，呈现文件模板：

```bash
GET /_render/template
{
  "file": "my_template",
  "params": {
    "status": [ "pending", "published" ]
  }
}
```

预注册模板可以使用以下呈现：

```bash
GET /_render/template/<template_name>
{
  "params": {
    "..."
  }
}
```

### Explain

你可以在运行模板时使用`Explain`参数：

```bash
GET _search/template
{
  "file": "my_template",
  "params": {
    "status": [ "pending", "published" ]
  },
  "explain": true
}
```

### Profiling

你可以在运行模板时使用`Profiling`参数：

```bash
GET _search/template
{
  "file": "my_template",
  "params": {
    "status": [ "pending", "published" ]
  },
  "profile": true
}
```