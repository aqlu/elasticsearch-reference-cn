# 文档API

本节开始将简短的介绍Elasticsearch的[数据复制模型]，将通过下面的CRUD API来更详细描述：

## 单文档API

* [Index API](./Document_APIS/Index_API.md)
* [Get API](./Document_APIS/Get_API.md)
* [Delete API](./Document_APIS/Delete_API.md)
* [Update API](./Document_APIS/Update_API.md)

## 多文档API

* [Multi Get API](./Document_APIS/Multi_Get_API.md)
* [Bulk API](./Document_APIS/Bulk_API.md)
* [Delete By Query API](./Document_APIS/Delete_By_Query_API.md)
* [Update By Query API](./Document_APIS/Update_By_Query_API.md)
* [Reindex API](./Document_APIS/Reindex_API.md)

> 注意
>
> 所有的单文档的CRUD API，`index`参数只能接受单一的索引库名称，或者是一个指向单一索引库的`alias`。