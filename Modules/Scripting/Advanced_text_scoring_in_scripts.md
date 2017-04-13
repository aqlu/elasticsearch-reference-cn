# 高级文本评分脚本

> 警告
> 本页面上所描述的功能被认为是实验性的、并且在未来的版本可被修改或删除

文本特征，譬如在脚本中通过访问`_index`变量来获取某一个指定词条的词频或文档频率。 它有可能被用在像这样的场景中，你想通过一个脚本在[function_core查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-function-score-query.html)中来实现一个自己的评分模块。在每个分片中统计文档集合，而不是每个索引。

## 名词解释

| 名词  |解释
| :---:|:-------------------------------
| df   |文档频率。词条在多少个文档中出现。按文档的每个字段计算。
| tf   |词频。词条在指定的文档中出现的次数。
| ttf  |总词频。词条出现在所有文档中的次数，也就是`tf`在所有文档中的和。按文档的每个字段计算。

`df`和`ttf`按每个分片计算，因此这些数字会跟随分片中的当前文档一起变化。


## 分片统计

_index.numDocs()

&emsp;&emsp;*分片中文档数*

_index.maxDoc()

&emsp;&emsp;*分片中最高的文档数*

_index.numDeletedDocs()

&emsp;&emsp;*分片中删除的文档数*


## 字段统计

字段统计可以通过下标运算这样访问：`_index['FIELD']`.

_index['FIELD'].docCount()

&emsp;&emsp;*含有 **FIELD** 字段的文档数。不包含已删除的文档。*

_index['FIELD'].sumttf()

&emsp;&emsp;* **FIELD** 字段的所有词条的词频总和。*

_index['FIELD'].sumdf()

&emsp;&emsp;* **FIELD** 字段的所有词条的文档频率和*

字段统计按每个分片计算，因此这些数字会跟随分片中的当前文档一起变化。在一个字段中词条的数量不能通过`_index`变量访问，请参考[Token count datatype](https://www.elastic.co/guide/en/elasticsearch/reference/current/token-count.html)章节解决。


## 词条统计

一个字段的词条统计可以通过下标运算这样访问：`_index['FIELD']['TERM']`。这将永远不会返回`null`，即使词条或字段不存在。如果你不需要词频，调用`_index['FIELD'].get('TERM', 0)` 避免不必要的频率计算。该标志将只影响你的文档设置[index_options](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-options.html)。

_index['FIELD']['TERM'].df()

&emsp;&emsp;* **FIELD**字段中**TERM**词条的文档频率。即使词条不存在文档中不存在也会有返回*

_index['FIELD']['TERM'].ttf()

&emsp;&emsp;* **FIELD**字段中**TERM**词条的总词频。即使词条不存在文档中不存在也会有返回。

_index['FIELD']['TERM'].tf()

&emsp;&emsp;* **FIELD**字段中**TERM**词条的词频。如果词条不存在于文档中，将返回0。


## 词条位置（positions），偏移（offsets）和有效载荷（payloads）

如果您需要获取一个词条在字段中的位置信息，可以调用`_index['FIELD'].get('TERM', flag)`来获取，`flag`可以是如下取值：

| &emsp;&emsp; | &emsp;&emsp;
| -------------|-------------------------------
|_POSITIONS    | 如果你需要这个词条的位置
|_OFFSETS      | 如果你需要这个词条的偏移
|_PAYLOADS     | 如果你需要这个词条的有效载荷
|_CACHE        | 如果你需要遍历所有位置好几次

迭代器使用底层的Lucene类遍历位置。出于效率的考虑，您只能遍历一次位置信息。如果您需要反复获取位置信息，可以设置`_CACHE`标志。

如果你需要获取多个信息，你可以用使用运算符`|`。例如，以下将返回一个对象的位置和有效载荷，以及所有的统计数据：

```js
_index['FIELD'].get('TERM', _POSITIONS | _PAYLOADS)
```

可以遍历返回的集合，通过一个`POS_OBJECT`对象来获取每个词条的位置、偏移量、有效载荷信息。

POS_OBJECT.position

&emsp;&emsp;*词条的位置。*

POS_OBJECT.startOffset

&emsp;&emsp;*词条开始的偏移量。*

POS_OBJECT.endOffset

&emsp;&emsp;*词条结束的偏移量。*

POS_OBJECT.payload

&emsp;&emsp;*词条的有效载荷。*

POS_OBJECT.payloadAsInt(missingValue)

&emsp;&emsp;*转换为整数的词条有效载荷。如果当前位置没有有效载荷时，**missingValue**将被返回。只能在你明确知道有效载荷为整数类型时才能调用此方法。*

POS_OBJECT.payloadAsFloat(missingValue)

&emsp;&emsp;*转换为浮点数的词条有效载荷。如果当前位置没有有效载荷时，**missingValue**将被返回。只能在你明确知道有效载荷为浮点类型时才能调用此方法。*

POS_OBJECT.payloadAsString()

&emsp;&emsp;*转换成字符串的词条有效载荷。如果当前位置没有有效载荷，`null`将被退回。只能在你明确知道有效载荷为字符串类型时才能调用此方法。*

例如，获取词条`foo`的有效载荷的总数：

```js
termInfo = _index['my_field'].get('foo',_PAYLOADS);
score = 0;
for (pos in termInfo) {
    score = score + pos.payloadAsInt(0);
}
return score;
```

## 词条向量：

`_index`变量只能被用于收集单一词条的统计数据。如果您想获取整个字段的所有词条信息，您必须先保存词条向量（参见[term_vector](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-vector.html)）。要访问它们，请调用`_index.termVectors()`得到[字段](https://lucene.apache.org/core/4_0_0/core/org/apache/lucene/index/Fields.html)实例, 你可以参见[lucene文档](https://lucene.apache.org/core/4_0_0/core/org/apache/lucene/index/Fields.html)来遍历这个集合，来获取每个词条在字段中的信息。如果词条向量没有被存储此方法将返回null。