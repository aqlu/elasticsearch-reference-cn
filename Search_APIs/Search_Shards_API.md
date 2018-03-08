# Search Shards API

search shards api返回将执行搜索请求的索引和分片。这可以提供有用的反馈，用于解决问题或使用`routing`和`shard perference`进行规划优化。当使用`aliase`过滤时，过滤器作为索引部分的一部分返回，在5.1.0中添加。

`index`和`type`参数可以是单个值，也可以逗号分隔。

 `type`参数在~~5.1.0~~废弃。

## 用法

完整示例：

```bash
GET /twitter/_search_shards
```
产生如下结果：

```bash
{
  "nodes": ...,
  "shards": [
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 0,
        "state": "STARTED",
        "allocation_id": {"id":"0TvkCyF7TAmM1wHP4a42-A"},
        "relocating_node": null
      }
    ],
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 1,
        "state": "STARTED",
        "allocation_id": {"id":"fMju3hd1QHWmWrIgFnI4Ww"},
        "relocating_node": null
      }
    ],
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 2,
        "state": "STARTED",
        "allocation_id": {"id":"Nwl0wbMBTHCWjEEbGYGapg"},
        "relocating_node": null
      }
    ],
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 3,
        "state": "STARTED",
        "allocation_id": {"id":"bU_KLGJISbW0RejwnwDPKw"},
        "relocating_node": null
      }
    ],
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 4,
        "state": "STARTED",
        "allocation_id": {"id":"DMs7_giNSwmdqVukF7UydA"},
        "relocating_node": null
      }
    ]
  ]
}
```

指定相同的请求，这次用一个`routing`值：

```bash
GET /twitter/_search_shards?routing=foo,baz
```

这将获得以下结果：

```bash
{
  "nodes": ...,
  "shards": [
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 0,
        "state": "STARTED",
        "allocation_id": {"id":"0TvkCyF7TAmM1wHP4a42-A"},
        "relocating_node": null
      }
    ],
    [
      {
        "index": "twitter",
        "node": "JklnKbD7Tyqi9TP3_Q_tBg",
        "primary": true,
        "shard": 1,
        "state": "STARTED",
        "allocation_id": {"id":"fMju3hd1QHWmWrIgFnI4Ww"},
        "relocating_node": null
      }
    ]
  ]
}
```
这次搜索只会针对两个分片执行，因为已经指定了路由值


## 所有参数

参数名   | 描述
-------|-----------
`routing` | 以逗号分隔的路由值列表考虑到当确定哪些碎片会反对执行的请求。
`preference` | 控制要对其执行搜索请求的分片副本的`perference`。 默认情况下，操作在分片副本之间随机化。 有关所有可接受值的列表，请参阅[preference](./Request_Body_Search/Preference.md)文档。
`local`     | 一个布尔值，是否在本地读取集群状态以便确定在何处分配碎片，而不是使用Master节点的集群状态。