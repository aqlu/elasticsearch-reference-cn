# 安装

Elasticsearch需要Java的最低版本的Java 8。在本文写作的时候，推荐使用Oracle JDK 1.8.0_73 版本。Java的安装，在各个平台上都有差异，所以我们不会在这里详细的介绍。在[Oracle官网](http://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html)可以找到Oracle的推荐安装文档。我想说的是，在安装Elasticsearch之前，你可以通过以下命令来检查你的Java版本（如果有需要，请安装或者升级）：

```bash
java -version
echo $JAVA_HOME
```

一旦Java安装完成， 我们就可以下载并安装 Elasticsearch 了。其二进制文件可以从[www.elasticsearch.org/download](http://www.elasticsearch.org/download)这里下载，你也可以从这里下载以前发布的版本。对于每个版本，你可以在`zip`、`tar`、`DEB`、`RPM`类型的包中选择下载。简单起见，我们使用`tar`包。

让我们通过如下指令开始下载Elasticsearch 5.3.0 tar包（Windows用户可能需要下载zip包）

```bash
curl -L -O https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.tar.gz
```

使用如下指令解压（Windows用户需要解压zip包）

```bash
tar -xvf elasticsearch-5.3.0.tar.gz
```

然后会在您当前目录中创建一堆文件和文件夹。然后转到 bin 目录中起，如下所示 : 

```bash
cd elasticsearch-5.3.0/bin
```

现在我们准备启动我们的节点，单个集群（Windows 用户应该运行 elasticsearch.bat 文件） :

```bash
./elasticsearch
```

如果一切顺利，你应该看到一堆看起来像下面的消息：

```js
[2016-09-16T14:17:51,251][INFO ][o.e.n.Node               ] [] initializing ...
[2016-09-16T14:17:51,329][INFO ][o.e.e.NodeEnvironment    ] [6-bjhwl] using [1] data paths, mounts [[/ (/dev/sda1)]], net usable_space [317.7gb], net total_space [453.6gb], spins? [no], types [ext4]
[2016-09-16T14:17:51,330][INFO ][o.e.e.NodeEnvironment    ] [6-bjhwl] heap size [1.9gb], compressed ordinary object pointers [true]
[2016-09-16T14:17:51,333][INFO ][o.e.n.Node               ] [6-bjhwl] node name [6-bjhwl] derived from node ID; set [node.name] to override
[2016-09-16T14:17:51,334][INFO ][o.e.n.Node               ] [6-bjhwl] version[5.3.0], pid[21261], build[f5daa16/2016-09-16T09:12:24.346Z], OS[Linux/4.4.0-36-generic/amd64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_60/25.60-b23]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [aggs-matrix-stats]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [ingest-common]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-expression]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-groovy]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-mustache]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [lang-painless]
[2016-09-16T14:17:51,967][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [percolator]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [reindex]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [transport-netty3]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded module [transport-netty4]
[2016-09-16T14:17:51,968][INFO ][o.e.p.PluginsService     ] [6-bjhwl] loaded plugin [mapper-murmur3]
[2016-09-16T14:17:53,521][INFO ][o.e.n.Node               ] [6-bjhwl] initialized
[2016-09-16T14:17:53,521][INFO ][o.e.n.Node               ] [6-bjhwl] starting ...
[2016-09-16T14:17:53,671][INFO ][o.e.t.TransportService   ] [6-bjhwl] publish_address {192.168.8.112:9300}, bound_addresses {{192.168.8.112:9300}
[2016-09-16T14:17:53,676][WARN ][o.e.b.BootstrapCheck     ] [6-bjhwl] max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
[2016-09-16T14:17:56,731][INFO ][o.e.h.HttpServer         ] [6-bjhwl] publish_address {192.168.8.112:9200}, bound_addresses {[::1]:9200}, {192.168.8.112:9200}
[2016-09-16T14:17:56,732][INFO ][o.e.g.GatewayService     ] [6-bjhwl] recovered [0] indices into cluster_state
[2016-09-16T14:17:56,748][INFO ][o.e.n.Node               ] [6-bjhwl] started
```

没有太多的详细信息，我们可以看到一个名称为`6-bjhwl`（在你的场景中可以不一样）的节点已启动，并且把它选举成主节点组成了一个集群。不用担心什么是主节点，重要的是我们已经在集群中启动了一个节点。

像前面说的那样，我们可以改写节点或集群的名字，可以在启动Elasticsearch的命令行中完成，就像这样：

```bash
./elasticsearch -Ecluster.name=my_cluster_name -Enode.name=my_node_name
```

另外需要注意到http那行关于HTTP的地址（192.168.8.112）和端口（9200）信息。默认情况下，Elasticsearch使用9200来提供对其REST API的访问。如果有必要，这个端口是可以配置的。