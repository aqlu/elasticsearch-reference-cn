# 重要的Elasticsearch配置

虽然Elasticsearch仅需要很少的配置，但在进入生产使用之前还是需要手动配置一些配置。

* [path.data和path.logs](#path)
* [cluster.name](#cluster-name)
* [node.name](#node-name)
* [bootstrap.memory_lock](#bootstrap-memory_lock)
* [network.host](#network-host)
* [discovery.zen.ping.unicast.hosts](#discovery-zen-ping-unicast-hosts)
* [discovery.zen.minimum_master_nodes](#discovery-zen-minimum_master_nodes)

## <span id="path">path.data与path.logs</span>

如果使用`.zip`或`.tar.gz`归档，则数据和日志目录是`$ES_HOME`的子文件夹。 如果这些重要的文件夹保留在其默认位置，则在将Elasticsearch升级到新版本时存在将其删除的高风险。

在生产使用中，几乎肯定要更改数据和日志文件夹的位置：

```yaml
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
```

RPM和Debian发行版已经使用数据和日志的自定义路径。

`path.data`配置可以设置为多个路径，在这种情况下，所有路径将用于存储数据（虽然属于单个分片的文件将全部存储在同一数据路径上）：

```yaml
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```

## <span id="cluster-name">cluster.name</span>

节点只能在群集与群集中的所有其他节点公共相同的`cluster.name`时才能加入群集。 默认名称为`elasticsearch`，但您应将其更改为描述集群用途的适当名称。

```yaml
cluster.name: logging-prod
```

确保不要在不同的环境中重复使用相同的集群名称，否则可能会导致加入错误集群的节点。

## <span id="node-name">node.name</span>

默认情况下，Elasticsearch将使用随机生成的uuid的前7个字符作为节点id。 请注意，节点ID是持久化的，并且在节点重新启动时不会更改，因此默认节点名称也不会更改。

值得配置一个更有意义的名称，这也将有重新启动节点后仍然存在的优势：

```yaml
node.name: prod-data-2
```

node.name也可以设置为服务器的HOSTNAME，如下所示：

```yaml
node.name: ${HOSTNAME}
```

## <span id="bootstrap-memory_lock">bootstrap.memory_lock</span>

禁用JVM的swapped交换到磁盘对节点健康是至关重要的。 实现的一种方法是将`bootstrap.memory_lock`设置为`true`。

要使此设置生效，需要首先配置其他系统设置。 有关如何正确设置内存锁定的更多详细信息，请参阅[启用bootstrap.memory_lock](./Bootstrap_Checks/Memory_lock_check.md)。


## <span id="network-host">network.host</span>

默认情况下，Elasticsearch仅仅绑定在本地回路地址——如：127.0.0.1与[::1]。这在一台服务器上跑一个开发节点是足够的。

> 提示
>
> 事实上，多个节点可以在单个节点上相同的`$ES_HOME`位置一天运行。这可以用于测试Elasticsearch形成集群的能力,但它不是一个推荐的配置方式用于生产。

为了将其它服务器上的节点形成一个可以相互通讯的集群，你的节点将不能绑定在一个回路地址上。 这里有更多的[网路配置](../Modules/Network_Settings.md)，通常你只需要配置`network.host`：

```yaml
network.host: 192.168.1.10
```

`network.host`也可以配置成一些能识别的特殊的值，譬如：`_local_`、`_site`、`_global_`，它们可以结合指定`:ip4`与`ip6`来使用。更多相信信息请参见：[网路配置](../Modules/Network_Settings.md#network-interface-values)

> 重要
>
> 一旦你自定义了`network.host`的配置，Elasticsearch将假设你已经从开发模式转到了生产模式，并将升级系统检测的警告信息为异常信息。更多信息请参见：[开发模式vs生产模式](./Important_System_Configuration.md#dev-vs-prod)

## <span id="discovery-zen-ping-unicast-hosts">discovery.zen.ping.unicast.hosts（单播发现）</span>

开箱即用，不用任何网络配置，Elasticsearch将绑定到可用的回路地址，并扫描9300年到9305的端口去连接同一机器上的其他节点,试图连接到相同的服务器上运行的其他节点。它提供了不需要任何配置就能自动组建集群的体验。

当与其它机器上的节点要形成一个集群时，你需要提供一个在线且可访问的节点列表。像如下来配置：

```yaml
discovery.zen.ping.unicast.hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 #①
   - seeds.mydomain.com #②
```

① 未指定端口时，将使用默认的`transport.profiles.default.port`值，如果此值也为设置则使用`transport.tcp.port`
_____________________________
② 主机名将被尝试解析成能解析的多个IP

## <span id="discovery-zen-minimum_master_nodes">discovery.zen.minimum_master_nodes</span>

为防止数据丢失，配置`discovery-zen-minimum_master_nodes`将非常重要，他规定了必须至少要有多少个`master`节点才能形成一个集群。

没有此设置时，一个集群在发生网络问题是可能会分裂成多个集群——脑裂——这将导致数据丢失。更多详细信息请参见：[通过`minimum_master_nodes`避免脑裂](../Modules/Node.md#split-brain)

为避免脑裂，你需要根据`master`节点数来设置法定人数：

```js
(master_eligible_nodes / 2) + 1
```

换句话说，如果你有三个`master`节点，最小的主节点数因被设置为`(3/2)+1`或者是`2`

```yaml
discovery.zen.minimum_master_nodes: 2
```