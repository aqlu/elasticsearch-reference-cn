# URL-based访问控制

许多用户使用具有基于URL的访问控制的代理来安全地访问Elasticsearch索引库。对于[multi-search](../Search_APIs/Multi_Search_API.md)、[multi-get](../Document_APIS/Multi_Get_API.md)和 [bulk](../Document_APIS/Bulk_API.md)请求，用户可以选择在 URL中和请求主体中的每个单独请求上指定索引。这可能使基于URL的访问控制具有挑战性。

要防止用户用户覆盖在URL中指定的索引，请将此设置添加到`elasticsearch.yml`文件中：

```yaml
rest.action.multi.allow_explicit_index: false
```

默认值是`true`，但是设置为`false`时， Elasticsearch将拒绝在请求主体中指定了明确索引的请求。
