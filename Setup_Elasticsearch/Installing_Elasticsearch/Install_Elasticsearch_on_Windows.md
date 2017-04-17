# Windows安装方式

Elasticsearch在Windows系统中可以采用`.zip`格式的安装包进行安装，这里有一个`elasticsearch-service.bat`的指令可以让Elasticsearch以服务进程的方式运行。

Elasticsearch的最新稳定版本可以在[Elasticsearch下载](https://www.elastic.co/downloads/elasticsearch)页面获取。其它版本可以在上[之前的下载页面](https://www.elastic.co/downloads/past-releases)找到。

> 注意
>
>Elasticsearch需要Java 8或更高版本。可以使用[Oracle官方发布](http://www.oracle.com/technetwork/java/javase/downloads/index.html)或开源版本的[OpenJDK](http://openjdk.java.net/)。

## 下载与安装`.zip`格式安装包

下载Elasticsearch v5.3.0的`.zip`压缩包：[https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.3.0.zip)

使用你的解压工具解压。它会创建一个`elasticsearch-5.3.0`目录，我们将用`%ES_HOME%`来引用它。在window的终端进入这个目录，像这样：

```batch
cd c:\elasticsearch-5.3.0
```

## 在命令行中运行Elasticsearch

Elasticsearch可以从命令行启动，如下所示：

```batch
.\bin\elasticsearch
```

默认情况下，Elasticsearch在前台运行，其打印日志采用标准输出（`stdout`），并且可以通过按下`Ctrl-C`停止。

## 在命令行配置Elasticsearch

Elasticsearch默认从`%ES_HOME%/config/elasticsearch.yml`文件中加载配置。这个配置文件的格式说明请参考[Elasticsearch配置](../Configuring_Elasticsearch.md)。

可以在配置文件中指定的任何配置都可以在命令行上指定，使用`-E`的语法，例如：

```batch
./bin/elasticsearch -Ecluster.name=my_cluster -Enode.name=node_1
```

> 注意
>
> 值如果包含空格，需要采用引号包围。例如：`Epath.logs="C:\My Logs\logs"`。

> 提示
>
> 通常情况下，任何集群范围的设置（例如`cluster.name`）应该被加入到elasticsearch.yml配置文件，而任何特定于节点的设置，例如`node.name`可以在命令行上被指定。

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

## <span id="windows-service">Windows后台服务方式安装</span>

Elasticsearch能够被安装到Window的后台服务，它可以在没有任何用户交互的情况下跟谁操作系统一起启动。这可以通过`bin\`目录中的`elasticsearch-service.bat`脚本来实现安装、删除、管理与配置后台服务的启动与停止，都是通过命令行来实现。

```batch
c:\elasticsearch-5.3.0\bin>elasticsearch-service

Usage: elasticsearch-service.bat install|remove|start|stop|manager [SERVICE_ID]

```

该脚本需要一个参数（要执行的命令），后面跟一个可选的服务ID名称（用于安装多个Elasticsearch服务时）。

可用的命令如下：

命令        | 描述
-----------|------------
install    | 安装Elasticsearch后台服务
remove     | 删除Elasticsearch后台服务（如果Elasticsearch后台服务已启动则先停止它）
start      | 启动Elasticsearch后台服务（如果已安装）
stop       | 停止Elasticsearch后台服务（如果已启动）
manager    | 启动一个图形界面来管理已安装的服务

根据现有的JDK/JRE（通过设置的`JAVA_HOME`），相应的64位（x64）或32位（x86）的服务将被安装。此信息是安装过程：

```batch
c:\elasticsearch-5.3.0\bin>elasticsearch-service install
Installing service      :  "elasticsearch-service-x64"
Using JAVA_HOME (64-bit):  "c:\jvm\jdk1.8"
The service 'elasticsearch-service-x64' has been installed.
```

> 注意
>
> 虽然JRE可用于Elasticsearch服务，由于其使用的客户机JVM（而不是JVM服务器模式，服务器模式对于长期运行应用提供了更好的性能），我们不鼓励使用JRE并会在使用时发出警告。

> 注意
>
> 升级（或降级）JVM的版本时，你并不需要重新安装Elasticsearch后台服务。但是如果升级跨JVM类型(譬如：从JRE到SE)时不行，这需要重新安装后台服务。

## 自定义服务设置

Elasticsearch服务可以在安装之前通过设置以下环境变量（可以通过[set命名](https://technet.microsoft.com/en-us/library/cc754250(v=ws.10).aspx)在命令行配置，也可以通过`系统设置`->`环境变量`的可视化界面配置）

参数名            | 描述
-----------------|------------------
SERVICE_ID       | 服务器名称，需要唯一。在同一机器上安装多个后台服务时使用。在32位系统里面默认为`elasticsearch-service-x86`，64位系统里面默认为`elasticsearch-service-x64`。
SERVICE_USERNAME | 运行的用户，默认为当前用户。
SERVICE_PASSWORD | 运行用户的密码。
SERVICE_DISPLAY_NAME| 服务名称。默认为`Elasticsearch <version> %SERVICE_ID%`
SERVICE_DESCRIPTION | 服务的描述。默认为`Elasticsearch <version> Windows Service - https://elastic.co`。
JAVA_HOME        | 自定义java路径。
LOG_DIR          | 日志目录，默认为`/var/log/elasticsearch`。
DATA_DIR         | Data目录，默认为`/var/lib/elasticsearch`。
CONF_DIR         | 配置文件目录（其中必须包括`elasticsearch.yml`和`log4j2.properties`文件），默认为`/etc/elasticsearch`。
ES_JAVA_OPTS     | 任何额外的JVM系统属性，你可能要应用。
ES_START_TYPE    | 启动模式。可以是`自动`或者`manual`（默认)
ES_STOP_TIMEOUT  | 优雅退出时，procrun 等待的秒数。默认为`0`

> 注意
>
> 核心的是，elasticsearch-service.bat依赖于[Apache Commons Daemon](http://commons.apache.org/proper/commons-daemon/)项目来安装服务。该服务在安装之前设置环境变量被复制，并会在服务生命周期中使用。这意味着服务安装完成之后所做的任何修改都不会生效，除非你重新安装它。

> 注意
>
> 在Windows中，[heap大小](../Important_System_Configuration/Set_JVM_heap_size_via_jvm.options.md)的调整可以使任何时候的命令行启动走设置，或者是第一次安装Elasticsearch后台服务的时候设置。要调整已安装服务的JVM堆大小，可使用服务管理器：`bin\elasticsearch-service.bat manager`

### 使用管理界面

`你也可以在安装完之后使用管理界面（elasticsearch-service-mgr.exe）来配置后台服务，它提供了查看与设置已安装服务的运行状态、启动类型、JVM、启动与停止等能力，简单地调用 elasticsearch-service.bat manager 在命令行会打开管理器窗口：`

![管理界面](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/service-manager-win.png)

通过管理界面做的大多数更改（如JVM设置），都要求服务重新启动才能生效。

## ZIP压缩文件的目录结构

`.zip`包是完全独立的。在`%ES_HOME%`中，只要解压压缩包之后会自动创建所有的文件和目录。

这是非常方便，因为开始使用Elasticsearch前你不需要创建任何目录，卸载Elasticsearch只需要删除`%ES_HOME%`目录。不过，建议你最好是修改下config目录、数据目录y与日志目录的默认位置，避免之后误删了重要的数据。

类型     |描述                        |默认位置                  |设置方式
--------|----------------------------|------------------------|----------
home    |Elasticsearch主目录或 %ES_HOME%|目录由解包归档创建        |
bin     |二进制脚本，包括启动节点的`elasticsearch`、安装插件的`elasticsearch-plugin`|%ES_HOME%/bin  |
conf    |配置文件，包括`lasticsearch.yml`|%ES_HOME%/config       |path.conf
data    |节点上分配的各索引/分片的数据文件的目录，可以配置多个位置。   |%ES_HOME%/data   |path.data
log     |日志文件的位置。               |%ES_HOME%/logs          |path.logs
plugins |插件的位置。每一个插件将被包含在一个子目录。|%ES_HOME%/plugins |
repo    |共享文件系统存储库位置。可以容纳多个位置。文件系统存储库可以放在这里指定的任意目录中的任何子目录。| 未配置 | path.repo
script  |脚本文件位置。                |%ES_HOME%/scripts        |path.scripts

## 下一步

现在，您搭建了一个测试环境Elasticsearch。开始更深入的研究或投入生产使用Elasticsearch之前，你需要做一些额外的配置：

* 了解如何[配置Elasticsearch](../Configuring_Elasticsearch.md)。
* 配置[重要的Elasticsearch设置](../Important_Elasticsearch_configuration.md)。
* 配置[重要的系统设置](../Important_System_Configuration.md)。
