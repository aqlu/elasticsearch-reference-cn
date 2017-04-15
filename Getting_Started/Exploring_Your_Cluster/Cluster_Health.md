# 集群健康(cluster health)

让我们以基本的健康检查作为开始，我们可以利用它来查看我们集群的状态。我们使用curl，当然你也可以使用任何可以创建HTTP/REST调用的工具，来使用该功能。
我们假设我们还在我们启动Elasticsearch的节点上并打开另外一个终端窗口。

要检查集群健康，我们将使用[_cat API](../../cat_APIs.md)。您可以在[Kibana’s Console](https://www.elastic.co/guide/en/kibana/master/console-kibana.html)点击 “VIEW IN CONSOLE” 或者点击 “COPY AS CURL” 链接然后粘贴到终端中使用 `curl` 中运行命令：

```js
curl '/_cat/health?v'
```

相应的响应是：

```js
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

可以看到，我们集群的名字是“elasticsearch”，正常运行，并且状态是绿色。

当我们查看集群状态的时候，我们可能得到绿色、黄色或红色三种状态。绿色代表一切正常（集群功能齐全）；黄色意味着所有的数据都是可用的，但是某些副本没有被分配（集群功能齐全）；红色则代表因为某些原因，某些数据不可用。注意，即使是集群状态是红色的，集群仍然是部分可用的（它仍然会利用可用的分片来响应搜索请求），但是可能你需要尽快修复它，因为你有丢失的数据。

从上面的响应中，我们可以看到 一共有一个节点，由于里面没有数据，我们有0个分片。注意，由于我们使用默认的集群名字（elasticsearch），并且由于Elasticsearch默认使用网络单播（unicast）发现同一机器上的其它节点，有可能您不小心在您的电脑上启动了多个节点，然后它们全部加入到了单个集群。在这种情形下，你可能在上面的响应中看到多个节点。

我们也可以获得节集群中的节点列表：

```js
curl '/_cat/nodes?v'
```

对应的响应是:

```js
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

这儿， 我们可以看到叫“PB2SGZY”的节点，这个节点是我们集群中的唯一节点。