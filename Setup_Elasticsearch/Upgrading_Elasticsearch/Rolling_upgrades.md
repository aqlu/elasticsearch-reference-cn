# 滚动升级

滚动升级允许用户每次升级Elasticsearch集群中的一个节点，终端用户感受不到任何停机。升级期间在同一个集群中会允许多个不同版本的Elasticsearch，正常生产期间不允许这样允许，因为新版本的分片副本将不会存放到老版本上。

查阅[此表](../Upgrading_Elasticsearch.md#upgrade-table)来验证你的Elasticsearch版本是否支持滚动升级。

执行滚动升级：

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

  1. **停止不必要的索引库和执行一个同步冲刷(可选)**

      你可以愉快地继续索引在升级。但是，如果你临时地关闭一些非不要的索引库以及执行一次[同步冲刷](../../Indices_APIs/Flush/Synced_Flush.md)请求可以帮助你快速恢复分片：

      ```js
      POST _flush/synced
      ```

      同步冲刷请求是一个”尽力而为“的操作。它可能会因为一些正在进行的索引操作而失败，但是如果有必要你可以反复的执行它，这是安全的。

  1. **<span id="upgrade-node">停止与升级一个节点</span>**

      在开始升级前停掉集群中其中的一个节点。

      > 提示
      >
      > 在使用`zip`或`tar`安装包时，`config`、`data`、`logs`、`plugin`目录默认放置在Elasticsearch的主目录下。
      >
      > 一个好的主意是将他们放置在其它位置，这样就不可能会在升级Elasticsearch时删除它们。自定义的路径可以通过`path.conf`、`path.logs`、`path.data`来[配置](../Important_Elasticsearch_configuration.md#path)，并且通过`ES_JVM_OPTIONS`来指定`jvm.options`文件的路径。
      >
      > [Debain](../Installing_Elasticsearch/Install_Elasticsearch_with_Debian_Package.md)与[RPM](../Installing_Elasticsearch/Install_Elasticsearch_with_RPM.md)安装包则会将这些目录放置在操作系统合适的地方。

      如果升级采用[Debain](../Installing_Elasticsearch/Install_Elasticsearch_with_Debian_Package.md)与[RPM](../Installing_Elasticsearch/Install_Elasticsearch_with_RPM.md)安装包：

        * 使用`rpm`与`dpkg`安装包升级时，所有的文件都会被放置在合适的位置，且配置文件不会被覆盖。

      如果升级采用`zip`或`tar`包：

        * 解压`zip`或`tar`包到一个新的目录，确保不会覆盖`config`或`data`目录。
        * 从老版本安装目录拷贝`config`目录到新的版本，或者使用`ES_JVM_OPTIONS`环境变量指定`jvm.options`文件与在命令行使用`-Epath.conf=`参数指向外部的目录。
        * 从老版本安装目录拷贝`data`目录到新的版本，或在`config/elasticsearch.yml`文件中配置`path.data`指向原来的数据文件目录。

  1. **升级所有的插件**

      升级Elasticsearch节点时必须升级插件。使用`elasticsearch-plugin`脚本安装你所有需要插件的正确版本。

  1. **启动升级的节点**

      现在开始启动升级的节点，并通过日志文件或如下命令来确认此节点能加入集群：

      ```js
      GET _cat/nodes
      ```

  1. **重新打开分片分配**

      ```js
      PUT _cluster/settings
      {
        "transient": {
          "cluster.routing.allocation.enable": "all"
        }
      }
      ```

  1. **等待节点的恢复**

      在升级下一个节点前你需要等待集群完成分片的分配工作。你可以通过[_cat/health](../../cat_APIs/cat_health.md)来检查完成得进度：

      ```js
      GET _cat/health
      ```

      一直等待`status`这一列的值从`yellow`变为`green`。状态`green`表示所有的主分片与副本分片都已分配完成。

      > 重要
      >
      > 在滚动升级期间，主分片如果被分配到高版本的节点，那么副本将不会被分配到低版本的节点。这是因为新版本可能是不同的数据格式，低版本不能识别它。
      >
      > 如果不能将分片分配到其它高版本的节点（譬如：集群中只有一个高版本的节点），副本分片将被遗留成未分配的分片，且集群状态会变成`yellow`。
      >
      > 遇到这种场景时，在开始后续的工作之前你需要检查下未初始化或未迁移的分片（在`init`与`relo`列中）。
      >
      > 一旦另一个节点升级，副本分片将会被分配且集群的健康状态将会变成`green`。

      分片没有被[同步冲刷](../../Indices_APIs/Flush/Synced_Flush.md)则可能还需要一些时间来恢复。单个分片的恢复状态可以通过[_cat/recovery](../../cat_APIs/cat_recovery.md)请求来监控：

      ```js
      GET _cat/recovery
      ```

      如果你停止新增索引，那在集群尽快完成恢复后重新开始新增索引是安全的做法。

  1. **重复以上步骤**

      当集群和节点恢复稳定后，所有剩余的节点重复以上步骤。