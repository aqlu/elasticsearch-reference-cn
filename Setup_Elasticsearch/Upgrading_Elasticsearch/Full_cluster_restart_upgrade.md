# 全集群重启升级

Elasticsearch在夸大版本（major）升级时需要重启整个集群。滚动升级不支持夸大版本。查阅[此表](../Upgrading_Elasticsearch.md#upgrade-table)来验证哪些是必须要全集群重启升级的。

执行全集群重启升级的过程如下：

  1. **禁用分片分配**

      当你关闭一个节点时，分配进程在等待一分钟之后开始将此节点上的分片复制到其它节点中，会造成很多浪费的I/O。这可以在节点关闭前通过禁用分片分配来避免：

      ```js
      PUT _cluster/settings
      {
        "transient": {
          "cluster.routing.allocation.enable": "none"
        }
      }
      ```

  1. **执行同步冲刷**

      你可以愉快地继续索引在升级。但是，如果你临时地关闭一些非不要的索引库以及执行一次[同步冲刷](../../Indices_APIs/Flush/Synced_Flush.md)请求可以帮助你快速恢复分片：

      ```js
      POST _flush/synced
      ```

      同步冲刷请求是一个”尽力而为“的操作。它可能会因为一些正在进行的索引操作而失败，但是如果有必要你可以反复的执行它，这是安全的。

  1. **停止与升级所有节点**

      在集群的所有节点上停掉Elasticsearch服务。在每一个节点上执行同样的步骤来完成[节点升级](./Rolling_upgrades.md#upgrade-node)。

  1. **升级所有的插件**

      升级Elasticsearch节点时必须升级插件。使用`elasticsearch-plugin`脚本安装你所有需要插件的正确版本。

  1. **启动集群**

      如果你有专门的master节点（在节点配置中设置`node.master`为`true`且`node.data`为`false`），先启动他们是一个好的主意。在处理数据节点之前等待它们形成一个集群并选举出一个主节点。你可以通过查看日志来检查进度。

      一旦对方发现了[最小的主节点数](../../Modules/Discovery/Zen_Discovery.md#master-election)，它们将在集群中选举主节点。从这时开始，就可以使用[_cat/health](../../cat_APIs/cat_health.md)与[_cat/nodes](../../cat_APIs/cat_nodes.md)API来监控节点加入集群：

      ```js
      GET _cat/health
      GET _cat/nodes
      ```

      使用这些API来检查所有节点已经成功地加入到集群。

  1. **等待状态`yellow`**

      一旦节点加入了集群，他将开始恢复本地存储的分片数据。刚开始[_cat/health](../../cat_APIs/cat_health.md)会返回`status`为`red`，这表示还有主分片未分配完成。

      一旦本地存储的分片恢复完成，`status`将会变成`yellow`，这表示所有主分片已恢复，但是副本分片没有分配。这是我们预料到的，因为分片分配被我们之前禁用了。

  1. **重新打开分片分配**

      延迟副本分片的分配，直到所有节点都加入了集群且完成了本地分片数据的分配。从这时开始，所有节点都已在集群中，重新打开分片分配是安全的：

      ```js
      PUT _cluster/settings
      {
        "transient": {
          "cluster.routing.allocation.enable": "all"
        }
      }
      ```

      集群将开始分片副本到所有的数据节点。这是你可以安全的开始新增索引与执行搜索操作，但是如果你在分片恢复之前延迟这些操作将会使得恢复过程变得更快。

      你可以使用[_cat/health](../../cat_APIs/cat_health.md)与[_cat/revocery](../../cat_APIs/cat_recovery.md)API来监控进度：

      ```js
      GET _cat/health
      GET _cat/recovery
      ```

      一旦`_cat/health`输出的`status`列变成`green`，所有主分片与副本分片都成功分配完成。