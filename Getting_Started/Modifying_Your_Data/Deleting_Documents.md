# 删除文档

删除文档是非常简单直接的。以下的例子展示了怎样删除ID为2的文档：

```js
DELETE /customer/external/2?pretty
```

可以通过[根据查询条件删除API](../../Document_APIS/Delete_By_Query_API.md)来一次删除符合指定条件的文档。如果要通过`根据查询条件删除API`来删除所有文档，使用删除整个索引库替代会效率更高。