# 脚本

脚本模块能够帮你通过脚本来自定义表达式求值。例如，你可以使用脚本作为搜索请求的一部分返回“脚本字段”或自定义一个评分查询。

默认脚本语言为[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)。您能够通过添加`lang`插件来启用其他语言编写的脚本。任何使用脚本的地方，您都可以通过设置`lang`参数来指定脚本的语言。

## 普通语言

这些语言可以用于任何的脚本API，并给予了最大的灵活性。

语言        |沙盒         |所需插件
-----------|:-----------:|---------------
painless   |yes          |内置
groovy     |no           |内置
javascript |no           |[lang-javascript](https://www.elastic.co/guide/en/elasticsearch/plugins/5.3/lang-javascript.html)
python     |no           |[lang-javascript](https://www.elastic.co/guide/en/elasticsearch/plugins/5.3/lang-javascript.html)

## 专业语言

这些语言不太灵活，但对于某些任务通常有更高的性能。

语言        |沙盒         |所需插件        |用途
-----------|:-----------:|---------------|------
[expression](./Scripting/Lucene_Expressions_Language.md)| yes       |内置   |快速自定义评分与排序
[mustache](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html)     |yes           |内置 | 模板
[java](./Scripting/Native_(Java)_Scripts.md) |n/a           |自己写 | 专业API


> **脚本与安全**（警告）
> 
> 语言在设计时是考虑到安全沙盒的。然而，非沙盒语言可能是一个安全问题，请阅读[脚本与安全](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-security.html)来获取详细信息。