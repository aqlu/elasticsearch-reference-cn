# 模块

本章节负责介绍Elasticsearch包含的各个模块的功能，每个模块的配置都可以通过如下方式进行配置：

<font color="#2b4590">*静态的*</font>

&emsp;&emsp;*这些配置项必须基于节点来进行设置，在启动节点前可以通过`elasticsearch.yml`配置文件、环境变量、命令行参数方式来进行配置。他们必须明确地在集群中的每个节点上进行设置。*

<font color="#2b4590">*动态的*</font>

&emsp;&emsp;*这些配置可以通过群集的[cluster-update-settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API进行动态更新。*

本节介绍的模块有：

[Cluster-level routing and shard allocation（集群级别的路由与分片分配）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cluster.html)）

&emsp;&emsp;*用来控制在何处、何时、以及如何给节点分配分片。*

[Discovery（发现）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery.html)

&emsp;&emsp;*集群中的节点彼此之间是如何发现的。*

[Gateway（网关）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-gateway.html)

&emsp;&emsp;*集群启动恢复前需要多少个节点加入。*

[HTTP](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html)

&emsp;&emsp;*用来控制配置`HTTP REST`接口。*

[Indices（索引）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-indices.html)

&emsp;&emsp;*所有跟索引相关的设置。*

[Network（网络）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)

&emsp;&emsp;*控制默认的网络设置。*

[Node client（节点客户端）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html)

&emsp;&emsp;*一个加入集群的Java客户端节点，但不能保存数据或作为主节点。*

[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)

&emsp;&emsp;*Elasticsearch内置的脚本语言，遵循尽可能的安全设计。*

[Plugins（插件）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-plugins.html)

&emsp;&emsp;*通过插件来扩展Elasticsearch的功能。*

[Scripting（脚本）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html)

&emsp;&emsp;*通过Lucene表达式、Groovy、Python、以及Javascript来自定义脚本。你也可以使用内置的脚本语言[Painless](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html)。*

[Snapshot/Restore（快照/还原）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html)

&emsp;&emsp;*通过快照与还原模块来备份你的数据。*

[Thread pools（线程池）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)

&emsp;&emsp;*Elasticsearch专用的线程池的信息。*

[Transport（传输）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-transport.html)

&emsp;&emsp;*Elasticsearch内部各节点之间的网络传输层通信配置。*

[Tribe nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-tribe.html)

&emsp;&emsp;*Tribe节点能加入一个或多个集群，并作为它们之间的联合客户端。*

[Cross cluster Search（跨集群搜索）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-cross-cluster-search.html)

&emsp;&emsp;*跨集群搜索功能可以通过一个不加入集群、并且能作为它们之间的联合客户端来实现一个以上集群的搜索。*