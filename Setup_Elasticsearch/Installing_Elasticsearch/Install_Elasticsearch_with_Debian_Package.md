# Debian软件包安装方式

Elasticsearch的Debian软件包可以从[我们的网站下载](#install-deb)或从[我们的APT仓库](#deb-repo)安装。它可以用于任何基于Debian的系统上安装Elasticsearch，如Debian和Ubuntu。

Elasticsearch的最新稳定版本可以在[Elasticsearch下载](https://www.elastic.co/downloads/elasticsearch)页面获取。其它版本可以在上[之前的下载页面](https://www.elastic.co/downloads/past-releases)找到。

> 注意
>
>Elasticsearch需要Java 8或更高版本。可以使用[Oracle官方发布](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或开源版本的[OpenJDK](http://openjdk.java.net/)。

## 导入Elasticsearch PGP Key

Elasticsearch的所有包都采用如下指纹与签名Key进行签名（PGP key [D88E42B4](https://pgp.mit.edu/pks/lookup?op=vindex&search=0xD27D666CD88E42B4)，可从[https://pgp.mit.edu](https://pgp.mit.edu/)）：

```bash
4609 5ACC 8548 582C 1A26 99A9 D27D 666C D88E 42B4
```

下载并安装该公用签名密钥：

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

## <span id="deb-repo">从APT库中安装</span>

在操作之前您可能需要在Debian系统中安装`apt-transport-https`软件包：

```bash
sudo apt-get install apt-transport-https
```

库定义保存到`/etc/apt/sources.list.d/elastic-5.x.list`：

```bash
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

> 注意
>
> 有以下几个原因不使用`add-apt-repository`：
> 1. `add-apt-repository`添加到系统`/etc/apt/sources.list`文件，而不是在`/etc/apt/sources.list.d`中清空之前的仓库文件
> 1. `add-apt-repository`需要按照一些非默认的依赖以及在许多发行版本中他不是默认安装的。
> 1. 老版本的`add-apt-repository`总是添加`deb-src`将导致错误，因为我们不提供源码包。如果你添加了`deb-src`，你将会看到一个如下的错误直到你删除了`deb-src`：
> ```
> Unable to find expected entry 'main/source/Sources' in Release file
> (Wrong sources.list entry or malformed file)
> ```

您可以通过如下方式安装Elasticsearch Debian软件包：

```bash
sudo apt-get update && sudo apt-get install elasticsearch
```

> 警告
>
> 如果同一Elasticsearch版本库中两个条目，在使用`apt-get update`是你会看到在这样的错误：
> ```
> Duplicate sources.list entry https://artifacts.elastic.co/packages/5.x/apt/ ...
> ```
> 在`/etc/apt/sources.list.d`和`/和/etc/apt/sources.list`文件中检查下重复的`/etc/apt/sources.list.d/elasticsearch-5.x.list`条目。

> 注意
>
> 在`systemd-based`的发行中，安装脚本会尝试设置内核参数（例如：`vm.max_map_count`）；你可以通过设置环境变量`ES_SKIP_SET_KERNEL_PARAMETERS=true`跳过此设置。

## <span id="install-deb">手动下载并安装Debian软件包</span>

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.deb
sha1sum elasticsearch-5.3.0.deb #①
sudo dpkg -i elasticsearch-5.3.0.deb
```

①  通过`sha1sum`或`shasum`比较官方发布的[SHA公钥](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.deb.sha1)


## `SysV init` vs `systemd`

Elasticsearch不是在安装后自动启动。如何启动和停止Elasticsearch取决于你的系统是否使用的`SysV init`或`system`（较新发行版中使用）。你可以说这是一个用来运行以下命令：

```bash
ps -p 1
```

## 通过`SysV init`启动Elasticsearch

使用`update-rc.d`命令来配置Elasticsearch在系统启动时自动启动:

```bash
sudo update-rc.d elasticsearch defaults 95 10
```

Elasticsearch可以通过`service`命令来启动与停止：

```bash
sudo -i service elasticsearch start
sudo -i service elasticsearch stop
```

任何原因的Elasticsearch启动失败，都会将原因打印到控制套。日志文件可以在`/var/log/elasticsearch/`中找到。


## 通过`systemd`启动Elasticsearch

要配置Elasticsearch在系统启动时自动启动，运行以下命令：

```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

Elasticsearch可以通过service命令来启动与停止：

```bash
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

无论Elasticsearch是否成功没有启动, 这些命令不提供反馈。相反，该信息将被写入位于`/var/log/elasticsearch/`的日志文件中。

默认情况下，Elasticsearch服务的信息不记录在信息`systemd` 的`journal`日志中。若要启用`journalctl`日志记录，在`elasticsearch.service`文件的`ExecStart`命令行中必须删除`--quiet`选项。

当`systemd`启用了日志记录，日志信息的使用可用`journalctl`命令：

`tail`查看`journal`:

```bash
sudo journalctl -f
```

要列出`journal`中elasticsearch服务的日志条目：

```bash
sudo journalctl --unit elasticsearch
```

要列出指定时间之后的列出`journal`中elasticsearch服务的日志条目：

```bash
sudo journalctl --unit elasticsearch --since  "2016-10-30 18:17:16"
```

更多的`journalctl`操作手册，请参考：[https://www.freedesktop.org/software/systemd/man/journalctl.html](https://www.freedesktop.org/software/systemd/man/journalctl.html)。

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

## 配置Elasticsearch

Elasticsearch默认从`/etc/elasticsearch/elasticsearch.yml`文件中加载配置。这个配置文件的格式说明请参考[Elasticsearch配置](../Configuring_Elasticsearch.md)。

Debian软件包也有一个系统配置文件（/etc/default/elasticsearch），它允许你设置以下参数：

参数名            | 描述
-----------------|------------------
ES_USER          | 运行的用户，默认为`elasticsearch`。
ES_GROUP         | 运行的Group，默认为`elasticsearch`。
JAVA_HOME        | 自定义java路径。
MAX_OPEN_FILES   | 最大的打开文件数，默认最大数量`65536`。
MAX_LOCKED_MEMORY| 最大锁定内存大小。你使用`bootstrap.memory_lock`的`elasticsearch.yml`选项将被设置为`unlimited`。
MAX_MAP_COUNT    | 进程的内存映射区域最大数量。如果你使用`mmapfs`的索引存储类型，确保此项设置为高值。欲了解更多信息，请查看[Linux内核文件](https://github.com/torvalds/linux/blob/master/Documentation/sysctl/vm.txt)查看相关的`max_map_count`。这是在启动elasticsearch之前通过`sysctl`设置的，默认为262144。
LOG_DIR          |日志目录，默认为`/var/log/elasticsearch`。
DATA_DIR         |Data目录，默认为`/var/lib/elasticsearch`。
CONF_DIR         |配置文件目录（其中必须包括`elasticsearch.yml`和`log4j2.properties`文件），默认为`/etc/elasticsearch`。
ES_JAVA_OPTS     |任何额外的JVM系统属性，你可能要应用。
RESTART_ON_UPGRADE|配置在安装包升级后重启，默认为`false`。这意味着你将在安装包后需要手动重新启动您的elasticsearch实例。这样做的原因是为了保证，在群集升级时不会连续的重新分配分片导致高网络流量、并降低群集的响应时间。

> 注意
>
> 使用systemd部署需要配置`systemd`的系统资源限制，而不是通过`/etc/sysconfig/elasticsearch`文件。更多信息请参见：[系统设置](../Important_System_Configuration/Configuring_system_settings.md#systemd)。

## Debian目录结构

Debian包中配置文件、日志和数据目录在Debian-based系统中对应的位置:

类型     |描述                        |默认位置                  |设置方式
--------|----------------------------|------------------------|----------
home    |Elasticsearch主目录或 $ES_HOME|/usr/share/elasticsearch   |
bin     |二进制脚本，包括启动节点的`elasticsearch`、安装插件的`elasticsearch-plugin`|/usr/share/elasticsearch/bin  |
conf    |配置文件，包括`elasticsearch.yml`|/etc/elasticsearch      |path.conf
conf    |环境变量，包括heap大小、文件操作符|/etc/default/elasticsearch|
data    |节点上分配的各索引/分片的数据文件的目录，可以配置多个位置。   |/var/lib/elasticsearch  |path.data
log     |日志文件的位置。               |/var/log/elasticsearch         |path.logs
plugins |插件的位置。每一个插件将被包含在一个子目录。|/usr/share/elasticsearch/plugins |
repo    |共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在这里指定的任意目录中的任何子目录。| 未配置 | path.repo
script  |脚本文件位置。                |/etc/elasticsearch/scripts       |path.scripts

## 下一步

现在，您搭建了一个测试环境Elasticsearch。开始更深入的研究或投入生产使用Elasticsearch之前，你需要做一些额外的配置：

* 了解如何[配置Elasticsearch](../Configuring_Elasticsearch.md)。
* 配置[重要的Elasticsearch设置](../Important_Elasticsearch_configuration.md)。
* 配置[重要的系统设置](../Important_System_Configuration.md)。
