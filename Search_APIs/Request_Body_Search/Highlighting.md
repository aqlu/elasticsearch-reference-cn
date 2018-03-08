# Highlighting

允许在一个或多个字段上高亮显示搜索结果。该实现使用了lucene的`plain` 、快速矢量（fvh）、`postings` 荧光笔。以下是搜索请求正文的示例：

```js
GET /_search
{
    "query" : {
        "match": { "content": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {}´
        }
    }
}
```

在上述情况下，`content`字段将为每个搜索命中的词高亮显示（每个搜索采样中将会有另一个元素，称为`highlight`，其中包括高亮字段和高亮片段）。

> 注意：
>
> 为了执行高亮显示，需要字段的实际内容。 如果有问题的字段被存储（在映射中存储设置为 true），它将被使用，否则，实际的 `_source` 将被加载，并且相关字段将从中提取。
> `_all` 字段不能从 `_source` 中提取，因此它只能用于高亮显示，如果它映射到将 `store` 设置为 `true`。

字段名称支持通配符符号。 例如，使用 `comment_ *` 将导致与表达式匹配的所有[text](../..//Mapping/Field_datatypes/Text_datatype.md)和[keyword](../..//Mapping/Field_datatypes/Keyword_datatype.md)字段（以及 5.0 之前的字符串）被高亮显示。 请注意，所有其他类型的字段将不会高亮显示。 如果您使用自定义映射器并要在字段上高亮显示，则必须显式提供字段名称。

## Plain highlighter（普通荧光笔）

荧光笔的默认选择是`plain`类型，并使用Lucene荧光笔。它试图在理解词重要性和短语查询中的任何词定位标准方面反映查询匹配逻辑。

> **警告：**
>
> 如果你想高亮很多文档中的大量字段与复杂的查询，这个荧光笔不会快。 在努力准确地反映查询逻辑，它创建一个微小的内存索引，并通过 Lucene 的查询执行计划程序重新运行原始查询条件，以获取当前文档的低级别匹配信息。 这对于每个字段和需要高亮显示的每个文档重复。 如果这在您的系统中出现性能问题，请考虑使用替代荧光笔。

## Postings highlighter

如果在索引的映射文件中设置`index_options`为`offsets`，则将使用`postings highlighter`来替代`plain highlighter`。 `postings highlighter`具有如下特性：

- 速度更快，因为它不需要重新分析要高亮显示的文本：文档越大，性能增益越好
- 比快速向量荧光笔所需的`term_vectors`需要更少的磁盘空间
- 将文本分成句子并高亮显示。非常适合自然语言，而不是与包含例如 html 标记的字段
- 将文档视为整个语料库，并使用 BM25 算法对单个句子进行评分，如同它们是该语料库中的文档

以下是一个在索引映射中设置内容字段的示例，以允许使用其上的 postings highlighter 来高亮显示：

```js
{
    "type_name" : {
        "content" : {"index_options" : "offsets"}
    }
}
```

> 注意：
>
> postings highlighter 指的是执行简单的查询术语高亮显示，而不考虑其位置。 这意味着，当与短语查询结合使用时，它将高亮显示查询所构成的所有术语，而不管它们是否实际上是查询匹配的一部分，从而有效地忽略了它们的位置。

> **警告：**
>
> postings highlighter 不支持高亮显示一些复杂的查询，例如类型设置为`match_phrase_prefix`的`match`查询。 在这种情况下，不会返回高亮显示的片段。

## Fast vector highlighter （快速矢量荧光笔）

如果通过在映射中将 term_vector 设置为 with_positions_offsets 来提供 term_vector 信息，则将使用快速向量荧光笔而不是普通荧光笔。 快速矢量荧光笔：

- 是更快，特别是对于大字段（> 1MB）
- 可以使用 boundary_chars，boundary_max_scan 和 fragment_offset 进行定制（见下文）
- 需要将 term_vector 设置为 with_positions_offsets，这会增加索引的大小
- 可以将多个字段的匹配合并为一个结果。请参阅 matched_fields
- 可以为不同位置的匹配分配不同的权重，以便在高亮显示促销词组匹配的 Boosting Query 时，可以将词组匹配排在匹配项上

下面是一个设置内容字段以允许使用快速向量荧光笔高亮显示的示例（这将导致索引更大）：

```js
{
    "type_name" : {
        "content" : {"term_vector" : "with_positions_offsets"}
    }
}
```

## Force highlighter type （指定荧光笔类型）

可以通过`type`字段允强制指定荧光笔类型。 这对于需要在启用`term_vectors`的字段上使用`plain`荧光笔时非常有用。 允许的值是：`plain`，`postings` 和 `fvh`。 以下是强制使用`plain`荧光笔的示例：

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"type" : "plain"}
        }
    }
}
```

## Force highlighting on source

强制高亮显示源上的高亮显示字段，即使字段单独存储。 默认为`false`。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"force_source" : true}
        }
    }
}
```

## Highlighting Tags

默认情况下，高亮的内容使用`<em>`与`</em>`标签包裹起来。他们可以通过`pre_tags`与`post_tags`来设置，例如：

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>"],
        "post_tags" : ["</tag1>"],
        "fields" : {
            "_all" : {}
        }
    }
}
```

使用快速向量荧光笔可以有更多的标签，“重要性”是有序的。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "_all" : {}
        }
    }
}
```

标签的样式可以内置在里面，下面是通过`pre_tags`设置了一个单一的样式标签：
```html
<em class="hlt1">, <em class="hlt2">, <em class="hlt3">,
<em class="hlt4">, <em class="hlt5">, <em class="hlt6">,
<em class="hlt7">, <em class="hlt8">, <em class="hlt9">,
<em class="hlt10">
```

和`</ em>`作为`post_tags`。 如果你认为更好的内置标签模式，只是发送电子邮件到邮件列表或打开一个问题。 以下是切换标记模式的示例：

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "tags_schema" : "styled",
        "fields" : {
            "content" : {}
        }
    }
}
```

## Encoder

编码器参数可用于定义高亮显示的文本的编码方式。 它可以是默认（无编码）或 html（将转义 html，如果你使用 html 高亮显示标签）。

## Highlighted Fragments
每个高亮显示的字段可以控制高亮的片段的大小（以字符为单位）（默认值为 100 ），以及要返回的最大片段数（默认值为 5 ）。例如：

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {"fragment_size" : 150, "number_of_fragments" : 3}
        }
    }
}
```

当使用`postings highlighter`时，`fragment_size`被忽略，因为它输出句子不考虑它们的长度。

除此之外，还可以指定高亮显示的片段需要按照分数排序：

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "order" : "score",
        "fields" : {
            "content" : {"fragment_size" : 150, "number_of_fragments" : 3}
        }
    }
}
```

如果`number_of_fragments`值设置为`0`，则不会生成片段，而是返回字段的整个内容，当然它会高亮显示。如果短文本（例如文档标题或地址）需要高亮显示，但不需要分段，这可能非常方便。请注意，在这种情况下会忽略 `fragment_size`。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "_all" : {},
            "bio.title" : {"number_of_fragments" : 0}
        }
    }
}
```

当使用`fvh`时，可以使用`fragment_offset`参数来控制从开始高亮显示的边距。

在没有匹配的片段高亮的情况下，默认是不返回任何东西。相反，我们可以通过将`no_match_size`（默认为`0`）设置为要返回的文本的长度，从字段的开头返回一段文本。实际长度可能比指定的短，因为它试图在单词边界上断开。当使用 `postings` 荧光笔时，不可能控制片段的实际大小，因此当 `no_match_size` 大于`0`时，第一个句子返回。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "fields" : {
            "content" : {
                "fragment_size" : 150,
                "number_of_fragments" : 3,
                "no_match_size": 150
            }
        }
    }
}
```

也可以通过设置`highlight_query`来高亮显示搜索查询之外的查询。 如果使用`rescore`查询，这是特别有用的，因为这些查询在默认情况下不会通过高亮显示来考虑。 Elasticsearch不会验证`highlight_query`以任何方式包含搜索查询，因此可以定义它，因此合法的查询结果根本不会高亮显示。 通常最好在`highlight_query`中包含搜索查询。 下面是在`highlight_query`中包含搜索查询和`rescore`查询的示例。

```js
GET /_search
{
    "stored_fields": [ "_id" ],
    "query" : {
        "match": {
            "content": {
                "query": "foo bar"
            }
        }
    },
    "rescore": {
        "window_size": 50,
        "query": {
            "rescore_query" : {
                "match_phrase": {
                    "content": {
                        "query": "foo bar",
                        "slop": 1
                    }
                }
            },
            "rescore_query_weight" : 10
        }
    },
    "highlight" : {
        "order" : "score",
        "fields" : {
            "content" : {
                "fragment_size" : 150,
                "number_of_fragments" : 3,
                "highlight_query": {
                    "bool": {
                        "must": {
                            "match": {
                                "content": {
                                    "query": "foo bar"
                                }
                            }
                        },
                        "should": {
                            "match_phrase": {
                                "content": {
                                    "query": "foo bar",
                                    "slop": 1,
                                    "boost": 10.0
                                }
                            }
                        },
                        "minimum_should_match": 0
                    }
                }
            }
        }
    }
}
```

注意，在这种情况下，文本片段的分数是由 Lucene 高亮显示框架计算的。 对于实现细节，您可以检查`ScoreOrderFragmentsBuilder.java`类。 另一方面，当使用过帐高亮显示器时，如上所述，使用BM25算法对分段进行打分。

## Global Settings

高亮设置可以在全局级别设置，然后在字段级别覆盖。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "number_of_fragments" : 3,
        "fragment_size" : 150,
        "fields" : {
            "_all" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] },
            "bio.title" : { "number_of_fragments" : 0 },
            "bio.author" : { "number_of_fragments" : 0 },
            "bio.content" : { "number_of_fragments" : 5, "order" : "score" }
        }
    }
}
```

## Require Field Match

`require_field_match`可以设置为`false`，这将导致任何字段被高亮显示，而不管查询是否与它们具体匹配。 默认行为是`true`，这意味着只有包含查询匹配的字段才会高亮显示。

```js
GET /_search
{
    "query" : {
        "match": { "user": "kimchy" }
    },
    "highlight" : {
        "require_field_match": false,
        "fields": {
                "_all" : { "pre_tags" : ["<em>"], "post_tags" : ["</em>"] }
        }
    }
}
```

## Boundary Characters

当使用快速向量荧光笔高亮显示字段时，可以配置 `boundary_chars`以定义什么构成用于高亮显示的边界。 它是一个单字符串，其中定义了每个边界字符。 它默认为`.,!? \t\n`。

`boundary_max_scan`允许控制查找边界字符的距离，默认值为`20`。

## Matched Fields

快速矢量荧光笔可以组合多个字段上的匹配，以使用 `matched_fields`高亮显示单个字段。 这对于以不同方式分析相同字符串的多字段来说是最直观的。 所有 `matched_fields`必须将`term_vector`设置为`with_positions_offsets`，但只会加载匹配的组合字段，因此只有该字段可以从`store`设置为`yes`时受益。

在下面的示例中，`content`由英语分析器分析，`content.plain`由标准分析器分析。

```js
GET /_search
{
    "query": {
        "query_string": {
            "query": "content.plain:running scissors",
            "fields": ["content"]
        }
    },
    "highlight": {
        "order": "score",
        "fields": {
            "content": {
                "matched_fields": ["content", "content.plain"],
                "type" : "fvh"
            }
        }
    }
}
```

以上匹配 “run with scissors” 和 “running with scissors”，并高亮显示 “running” 和 “scissors”，但不是 “run”。 如果两个短语出现在一个大的文档中，则 “running with scissors” 在片段列表中的 “run with scissors” 上排序，因为该片段中有更多匹配项。

```js
GET /_search
{
    "query": {
        "query_string": {
            "query": "running scissors",
            "fields": ["content", "content.plain^10"]
        }
    },
    "highlight": {
        "order": "score",
        "fields": {
            "content": {
                "matched_fields": ["content.plain"],
                "type" : "fvh"
            }
        }
    }
}
```

面的查询不会高亮显示“run”或“scissor”，但显示仅列出匹配字段中匹配组合的字段（内容）就足够了。

> 注意：
>
> 从技术上讲，向匹配字段添加不共享与匹配组合的字段相同的基础字符串的字段也是很好的做法。结果可能没有多大意义，如果其中一个匹配不在文本的末尾，那么整个查询将失败。

> 注意：
>
> 将`matched_fields`设置为非空数组会涉及少量开销，因此总是喜欢将
> ```js
> "highlight": {
>     "fields": {
>       "content": {}
>   }
> }
>```
> 到
> ```js
> "highlight": {
>     "fields": {
>      "content": {
>             "matched_fields": ["content"],
>             "type" : "fvh"
>         }
>     }
> }
> ```

## Phrase Limit

快速向量荧光笔有一个`phrase_limit`参数，阻止它分析太多的短语和吃大量的内存。 它默认为`256`，所以只有文档中前 `256`个匹配的短语被考虑。 您可以使用`phrase_limit`参数提高限制，但请记住，评分更多的短语会消耗更多的时间和内存。

如果使用`matched_fields`，请记住每个匹配字段的 `phrase_limit`短语会被考虑。

## Field Highlight Order

Elasticsearch 按照它们发送的顺序高亮显示字段。 每个 json spec 对象是无序的，但如果你需要明确的字段的高亮显示的顺序，你可以使用数组的字段，如：

```js
 "highlight": {
     "fields": [
         {"title":{ /*params*/ }},
         {"text":{ /*params*/ }}
     ]
 }
```

没有一个内置于 Elasticsearch 的荧光笔关心字段高亮显示的顺序，但插件可能。
