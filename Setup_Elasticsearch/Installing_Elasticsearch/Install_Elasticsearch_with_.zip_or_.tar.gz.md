# .zip或.tar.gz文件的安装方式

Elasticsearch提供了`.zip`与`.tar.gz`的安装包，这些包可用于任何系统上安装Elasticsearch，是一种简单的安装包格式。

Elasticsearch的最新稳定版本可以在[Elasticsearch下载](https://www.elastic.co/downloads/elasticsearch)页面获取。其它版本可以在上[之前的下载页面](https://www.elastic.co/downloads/past-releases)找到。

> 注意
>
>Elasticsearch需要Java 8或更高版本。可以使用[Oracle官方发布](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或开源版本的[OpenJDK](http://openjdk.java.net/)。

## 下载与安装`.zip`格式安装包

`.zip`压缩包的Elasticsearch v5.3.0版本下载和安装方式如下：

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip
sha1sum elasticsearch-5.3.0.zip  #①
unzip elasticsearch-5.3.0.zip
cd elasticsearch-5.3.0/ #②
```

①  通过`sha1sum`或`shasum`比较官方发布的[SHA公钥](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip.sha1)
______________________________
②  该目录被作为`$ES_HOME`


## 下载与安装`.tar.gz`格式安装包

`.tar.gz`压缩包的Elasticsearch v5.3.0版本下载和安装方式如下：

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.tar.gz
sha1sum elasticsearch-5.3.0.tar.gz #①
tar -xzf elasticsearch-5.3.0.tar.gz
cd elasticsearch-5.3.0 / #②
```

①  通过`sha1sum`或`shasum`比较官方发布的[SHA公钥](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip.sha1)
______________________________
②  该目录被作为`$ES_HOME`


## 在命令行中运行Elasticsearch

Elasticsearch可以从命令行启动，如下所示：

```bash
./bin/elasticsearch
```

默认情况下，Elasticsearch在前台运行，其打印日志采用标准输出（`stdout`），并且可以通过按下`Ctrl-C`停止。

> 注意
>
> 所有脚本包的运行都依赖于`Bash`的版本，`Bash`需要支持数组指令且假设它在`/bin/bash`是可用的。所以`Bash`需要在此目录中或者通过软连接放置在此目录。

## 检查Elasticsearch运行

您可以在已运行的Elasticsearch节点上，发送一个HTTP请求测试`localhost`的`9200`端口：

```js
GET /
```

返回的消息应该是这样的：

```js
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "5.3.0",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "6.4.1"
  },
  "tagline" : "You Know, for Search"
}
```

在命令行使用`-q`或`--quiet`选项可以关闭在`stdout`上的日志打印。

## 作为守护程序运行

在命令行使用`-d`选项可以使Elasticsearch作为一个守护进程在后台运行，使用`-p`选项可以记录进程ID到一个文件：

```bash
./bin/elasticsearch -d -p pid
```

日志信息可以在`$ES_HOME/logs/`目录中找到。

要关闭Elasticsearch，可以根据`pid`文件中的记录来`kill`它：

```bash
kill `cat pid`
```

> 注意
>
> 在[RPM](./Install_Elasticsearch_with_RPM.md)和[Debian](./Install_Elasticsearch_with_Debian_Package.md)格式的安装包中，提供了快速脚步来启动和停止Elasticsearch进程。

## 在命令行配置Elasticsearch

Elasticsearch默认从`$ES_HOME/config/elasticsearch.yml`文件中加载配置。这个配置文件的格式说明请参考[Elasticsearch配置](../Configuring_Elasticsearch.md)。

可以在配置文件中指定的任何配置都可以在命令行上指定，使用`-E`的语法，例如：

```bash
./bin/elasticsearch -d -Ecluster.name=my_cluster -Enode.name=node_1
```

> 提示
>
> 通常情况下，任何集群范围的设置（例如`cluster.name`）应该被加入到elasticsearch.yml配置文件，而任何特定于节点的设置，例如`node.name`可以在命令行上被指定。

## `.zip`和`.tar.gz`的文档目录结构

`.zip`和`.tar.gz`包是完全独立的。在`$ES_HOME`中，只要解压压缩包之后会自动创建所有的文件和目录。

这是非常方便，因为开始使用Elasticsearch前你不需要创建任何目录，卸载Elasticsearch只需要删除`$ES_HOME`目录。不过，建议你最好是修改下config目录、数据目录y与日志目录的默认位置，避免之后误删了重要的数据。

类型     |描述                        |默认位置                  |设置方式
--------|----------------------------|------------------------|----------
home    |Elasticsearch主目录或 $ES_HOME|目录由解包归档创建        |
bin     |二进制脚本，包括启动节点的`elasticsearch`、安装插件的`elasticsearch-plugin`|$ES_HOME/bin  |
conf    |配置文件，包括`lasticsearch.yml`|$ES_HOME/config       |path.conf
data    |节点上分配的各索引/分片的数据文件的目录，可以配置多个位置。   |$ES_HOME/data   |path.data
log     |日志文件的位置。               |$ES_HOME/logs          |path.logs
plugins |插件的位置。每一个插件将被包含在一个子目录。|$ES_HOME/plugins |
repo    |共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在这里指定的任意目录中的任何子目录。| 未配置 | path.repo
script  |脚本文件位置。                |$ES_HOME/scripts        |path.scripts

## 下一步

现在，您搭建了一个测试环境Elasticsearch。开始更深入的研究或投入生产使用Elasticsearch之前，你需要做一些额外的配置：

* 了解如何[配置Elasticsearch](../Configuring_Elasticsearch.md)。
* 配置[重要的Elasticsearch设置](../Important_Elasticsearch_configuration.md)。
* 配置[重要的系统设置](../Important_System_Configuration.md)。