# Elasticsearch升级

> 重要
>
> 开始升级Elasticsearch之前：
> * 查询[破坏性的改变](./Breaking_changes.md)文档
> * 升级前使用[Elasticsearch迁移插件](https://github.com/elastic/elasticsearch-migration/)来检测潜在的问题。
> * 在升级生产环境之前先在开发环境做升级测试
> * 总是在升级前[备份你的数据](../Modules/Snapshot_And_Restore.md)，你**不能回滚**到早期版本，除非你有备份数据。
> * 如果你使用自己定义插件，请检查它可用的兼容版本。

Elasticsearch通过可以通过滚动升级来不中断服。本节详细介绍如何执行滚动升级和集群重启升级。

确定是否支持滚动升级你的版本,请查阅<span id="upgrade-table">此表</span>:

从哪个版本升级 | 升级到哪个版本 | 支持升级的类型
------------|--------------|-----------------
1.x         |5.x           | [索引重建升级](./Upgrading_Elasticsearch/Reindex_to_upgrade.md)
2.x         |2.y           | [滚动升级](./Upgrading_Elasticsearch/Rolling_upgrades.md)(当y>x时)
2.x         |5.x           | [全集群重启升级](./Upgrading_Elasticsearch/Full_cluster_restart_upgrade.md)
5.0.0 pre GA|5.x           | [全集群重启升级](./Upgrading_Elasticsearch/Full_cluster_restart_upgrade.md)
5.x         |5.y           | [滚动升级](./Upgrading_Elasticsearch/Rolling_upgrades.md)(当y>x时)

> 重要
>
> 在1.x或之前创建的索引
>
> Elasticsearch只能阅读**上一个major版本**创建的索引。例如：5.x只能阅读2.x创建的索引，不能阅读1.x创建的索引。
>
> 这个条件也被应用在[快照与还原](../Modules/Snapshot_And_Restore.md)。如果一个索引库最初是由1.x的版本创建的他将不能还原到5.x的版本，即使快照是由2.x的集群创建。
>
> Elasticsearch 5.x的节点在存在太老的索引库时将无法启动。
>
> 参见[索引重建升级](./Upgrading_Elasticsearch/Reindex_to_upgrade.md)来获取更多如何升级老索引库的相关信息。