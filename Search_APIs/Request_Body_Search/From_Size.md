# From / Size

结果的分页可以通过使用`from`和`size`参数来完成。 `from`参数定义了您要提取的第一个结果的偏移量。 `size`参数允许您配置要返回的最大匹配数。

虽然`from`和`size`可以设置为请求参数，但它们也可以在搜索正文中设置。`from`默认值为`0`，`size`默认为`10`。

```js
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
```

注意`from + size`不能超过索引设置的`index.max_result_window`，默认为`10000`。 有关深入滚动的更有效方法，请参阅[Scroll](./Scroll.md)或[Search After](Search_After.md) API。
