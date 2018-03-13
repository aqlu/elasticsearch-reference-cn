# Percolator

> **警告**
>
> ## 5.0.0已废弃
> `Percolate`与`multi percolate` APIs已废弃，请使用新的[percolate query](../Query_DSL/Specialized_queries/Percolate_Query.md)

对于版本`5.0.0-alpha1`或之后创建的索引，过滤器自动将查询条件与过滤器查询进行索引。这使得渗滤器能够更快速地渗透文件。建议重新创建任何`5.0.0`之前的索引库以利用这一新的优化。