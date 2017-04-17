# 配置Elasticsearch

Elasticsearch提供了不错的默认值，且仅需较少的配置。 可以使用[群集的更新设置API](../Cluster_APIs/Cluster_Update_Settings.md)在正在运行的群集上更改大多数配置。

配置文件中可能需要包含特定于节点的配置（例如：`node.name`和`paths`）与节点为了能够加入群集而需要的配置，例如：`cluster.name`和`network.host`。

## 配置文件位置

Elasticsearch有两个配置文件：

* 用于配置Elasticsearch的`elasticsearch.yml`；
* log4j2.properties用于配置Elasticsearch日志记录的`log4j2.properties`。

这些文件位于配置文件目录中，其位置默认为`$ES_HOME/config/`。Debian和RPM软件包将config目录位置设置为`/etc/elasticsearch/`。
可以使用`path.conf`设置更改配置文件目录的位置，如下所示：

```bash
./bin/elasticsearch -Epath.conf=/path/to/my/config/
```

## 配置文件格式

配置格式为[YAML](http://www.yaml.org/)。 下面是更改数据目录和日志目录路径的示例：

```yaml
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

设置也可以展平开如下：

```yml
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

## 环境变量替换

配置文件中可以使用`${...}`符号来引用环境变量的值，例如：

```yml
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

## 交互式配置

对于您不希望保存在配置文件中的设置，您可以使用`${prompt.text}`或`${prompt.secret}`这样的值并在控制台启动Elasticsearch。`${prompt.secret}`禁用了打印，所以输入的值不会在终端显示; `${prompt.text}`将允许您在键入时看到值。例如：

```yml
node:
  name: ${prompt.text}
```

当启动Elasticsearch时，将提示您进行输入，如下所示：

```bash
Enter value for [node.name]:
```

> 注意
>
> 如果在配置文件中使用`${prompt.text}`或`${prompt.secret}`，Elasticsearch将不能作为服务进程运行或者是在后台运行。

## 设置默认配置

可以在命令行中设置新的默认配置。这将设置一个默认值，除非在配置文件中指定另一个值。

例如，如果Elasticsearch按如下方式启动：

```bash
./bin/elasticsearch -Edefault.node.name=My_Node
```

`node.name`的值将为`My_Node`，除非你在命令行覆写了`es.node.name`或者是在配置文件中设置了`node.name`。

## 日志配置

Elasticsearch使用[Log4j2](https://logging.apache.org/log4j/2.x/)进行日志记录。 Log4j2可以通过log4j2.properties文件进行配置。Elasticsearch公开了三个属性：`${sys:es.logs.base_path}`、`${sys:es.logs.cluster_name}`、`${sys:es.logs.node_name}`（如果节点名字通过`node.name`明确配置过），这三个属性可以在配置文件中引用，以确定日志文件的存放路径; `${sys:es.logs.base_path}`被解析为日志目录，`${sys:es.logs.cluster_name}`被解析为集群名称（默认用作日志文件名的前缀），`${sys:es.logs.node_name}`被解析为节点名字（如果节点名字明确配置过）。

例如，如果您的日志目录（`path.logs`）是`/var/log/elasticsearch`，并且您的集群名为`production`，那么`${sys:es.logs}`将解析为`/var/log/elasticsearch/production`，`${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log`将被解析为`/var/log/elasticsearch/production.log`。

```properties
appender.rolling.type = RollingFile   #①
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs}.log   #②
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %.10000m%n
appender.rolling.filePattern = ${sys:es.logs}-%d{yyyy-MM-dd}.log   #③
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy   #④
appender.rolling.policies.time.interval = 1    #⑤
appender.rolling.policies.time.modulate = true    #⑥
```

① 配置记录器为`RollingFile`
_________________________________________
② 记录到`/var/log/elasticsearch/production.log`
_________________________________________
③ 滚动记录日志记录到`/var/log/elasticsearch/production-yyyy-MM-dd.log`
_________________________________________
④ 使用基于时间的滚动策略
_________________________________________
⑤ 每日滚动日志
_________________________________________
⑥ 在日边界对齐卷（而不是每二十四小时滚动一次）

如果将`.gz`或`.zip`附加到`appender.rolling.filePattern`，那么日志将在滚动时压缩。

如果你想保留指定时间段的日志，可以使用一个带有删除动作的滚动策略。

```properties
appender.rolling.strategy.type = DefaultRolloverStrategy #①
appender.rolling.strategy.action.type = Delete #②
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path} #③
appender.rolling.strategy.action.condition.type = IfLastModified #④
appender.rolling.strategy.action.condition.age = 7D #⑤
appender.rolling.strategy.action.PathConditions.type = IfFileName #⑥
appender.rolling.strategy.action.PathConditions.glob = ${sys:es.logs.cluster_name}-* #⑦
```

① 配置滚定处理器DefaultRolloverStrategy
_________________________________________
② 为滚动滚定处理器配置删除动作
_________________________________________
③ 日志文件目录
_________________________________________
④ 指定滚定的条件
_________________________________________
⑤ 保留日志的时间
_________________________________________
⑥ 根据文件名匹配，仅删除超过7天的文件
_________________________________________
⑦ 根据`${sys:es.logs.cluster_name}-*`格式去匹配删除文件; 它仅仅只删除Elasticsearch的日志，不会删除`deprecation`与`slow`的日志。

可以加载多个配置文件（在这种情况下，它们将被合并），只要它们命名为`log4j2.properties`并将Elasticsearch config目录作为父目录; 这对于暴露其他日志记录器的插件很有用。 日志部分包含java包及其对应的日志级别。 记录器部分包含日志的目标。 有关如何自定义日志记录和所有支持的追加器的详细信息可以在[Log4j文档](http://logging.apache.org/log4j/2.x/manual/configuration.html)中找到。

## Deprecation（过期）日志

除了常规日志记录之外，Elasticsearch还允许您启用日志来记录一些过期的操作。 例如，这允许您在早期就确定您将需要在未来迁移哪些功能。 默认情况下，过期日志会开启并以WRAN级别记录，此级别会记录所有过期操作的日志。

```properties
logger.deprecation.level = warn
```

它将在日志目录中创建每日滚动的deprecation日志文件。 定期检查此文件，特别是当您打算升级到新的主版本。

默认的日志配置已为弃用日志设置了滚动策略，将在1GB之后滚动和压缩，并且最多保留五个日志文件（四个已滚动的日志和一个正在记录的日志）。

您可以通过在`config/log4j2.properties`文件中设置deprecation日志级别设置为`error`来禁用它。