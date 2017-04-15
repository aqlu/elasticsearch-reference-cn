# Docker安装方式

Elasticsearch也提供了可用的Docker镜像，镜像包含了[X-Pack](https://www.elastic.co/guide/en/x-pack/5.3/index.html)。

## 安全提示

> 提示
>
> [X-Pack](https://www.elastic.co/guide/en/x-pack/5.3/index.html)被在此镜像中预装。请花几分钟时间来熟悉[X-Pack安全](https://www.elastic.co/guide/en/x-pack/5.3/security-getting-started.html)和如何更改默认密码。默认用户`elastic`的密码为`changeme`

> 提示
>
> X-Pack包含了30天的体验License，在这之后你需要选择[激活购买](https://www.elastic.co/subscriptions)或[禁用安全插件](https://www.elastic.co/guide/en/x-pack/5.3/security-settings.html)。基础的License是免费的且包含了[监控](https://www.elastic.co/products/x-pack/monitoring)插件。

获取Elasticsearch镜像只需要一个简单的`docker pull`指令就可以从Elastic Docker仓库中得到。

如下命令演示了提取Docker镜像：

```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:5.3.0
```

## 在命令行中运行Elasticsearch

### 开发模式

Elasticsearch可以通过下面的指令快起的在开发或测试环境中启动：

```bash
docker run -p 9200:9200 -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" docker.elastic.co/elasticsearch/elasticsearch:5.3.0
```

### <span id="prpduction-mode">生产模式</span>

> 重要提示
>
> 用于生产`vm_max_map_count`内核参数需要被设置到至少262144。不同平台的设置方式:
>
> * Linux
>
> &emsp;&emsp;`vm_max_map_count`参数需要永久的配置在`/etc/sysctl.conf`中：
>
> ```bash
> $ grep vm.max_map_count /etc/sysctl.conf
> vm.max_map_count=262144
> ```
>
> &emsp;&emsp;正在运行的系统实时生效可使用：`sysctl -w vm.max_map_count=262144`
>
> * OSX with [Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac)
>
> &emsp;&emsp;`vm_max_map_count`参数必须要在`xhyve`虚拟机中配置：
>
> ```bash
> $ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
> ```
>
> &emsp;&emsp;登录到root用户下，像Linux那样通过`sysctl`来配置
>
> ```bash
> sysctl -w vm.max_map_count=262144
> ```
>
> * OSX with [Docker Toolbox](https://docs.docker.com/engine/installation/mac/#docker-toolbox)
>
> &emsp;&emsp;`vm_max_map_count`参数必须要在`docker-machine`中配置：
>
> ```bash
> docker-machine ssh
> sudo sysctl -w vm.max_map_count=262144
> ```

下面的示例演示了启动一个包含两个节点的集群。启动集群之前，你需要编写好[docker-compose.yml](#docker-compose)，然后输入：

```bash
docker-compose up
```

> 注意
>
> 如果Linux上没有预安装`docker-compose`指令，请参考此站点进行安装：[docker-compose](https://docs.docker.com/compose/install/#install-using-pip)。

`elasticsearch1`节点将会监听`localhost:9200`，并且`elasticsearch1`与`elasticsearch2`将会通过Docker网络通信。

这个例子还使用[Docker named volumes](https://docs.docker.com/engine/tutorials/dockervolumes)，被称为esdata1和esdata2,，如果不存在会先创建。

<span id="docker-compose">`docker-compose.yml:`</span>

```yaml
version: '2'
services:
  elasticsearch1:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.3.0
    container_name: elasticsearch1
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    mem_limit: 1g
    cap_add:
      - IPC_LOCK
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.3.0
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch1"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    mem_limit: 1g
    cap_add:
      - IPC_LOCK
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
    driver: bridge
```

若要停掉整个集群，输入`docker-compose down`即可。数据目录会被保留下来，所以再次通过`docker-compose up`启动整个集群会得到之前的数据。如要停止整个集群且删除之前的数据，请使用`docker-compose down -v`即可。

### 检查集群状态

```bash
curl -u elastic http://127.0.0.1:9200/_cat/health
Enter host password for user 'elastic':
1472225929 15:38:49 docker-cluster green 2 2 4 2 0 0 0 0 - 100.0%
```

日志信息将被输出到控制台且被Docker日志驱动处理。默认情况下你可以通多`docker logs`来获取日志信息。

## <span id="configuration-method">在Docker中配置Elastcsearch</span>

Elasticsearch从`/usr/share/elasticsearch/config`文件中加载配置。这些配置文件的文档在[Elasticsearch设置](../Configuring_Elasticsearch.md)与[JVM设置](../Important_System_Configuration/Configuring_system_settings.md#jvm-options)。

镜像通过了配置多种方式，传统的是修改`elasticsearch.yml`文件，但也可以通过环境变量来设置参数：

### A.通过Docker环境变量方式设置

例如，在使用`docker run`的时候输入`-e "cluster.name=mynewclustername"`来定义集群名称。双引号是必须的。

> 注意
>
> [默认设置](../Configuring_Elasticsearch.md#_setting_default_settings)与普通设置有一些区别。如果你定义了，模板中以`default.`开头的普通设置将不会被覆盖。

### B.绑定挂载文件方式设置

创建一个自定义的配置文件并挂载到镜像中配置文件同样的路径。例如，使用`docker run`时绑定挂载一个`custom_elasticsearch.yml`的参数：

```bash
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

> 重要
>
> `custom_elasticsearch.yml`需要能够被`uid:gid 1000:1000`读取。

### C.自定义镜像

在某些环境中，你可以需要自定义镜像包括你的配置。一个完整的`Dockerfile`可能像是下面这个样子：

```docker
FROM docker.elastic.co/elasticsearch/elasticsearch:5.3.0
ADD elasticsearch.yml /usr/share/elasticsearch/config/
USER root
RUN chown elasticsearch:elasticsearch config/elasticsearch.yml
USER elasticsearch
```

然后您可以像这样尝试构建：

```bash
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

### D.覆盖镜像默认的命令行参数

可以通过镜像的默认指令来覆盖Elsticsearch提供的命令行参数，例如：

```bash
docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername
```

## 生产使用以及默认值的注意事项

我们收集了一些生产使用的最佳实践配置。

> 注意
>
> 所有提及到的参数都假定你使用`docker run`。

1. 通过Docker CLI设置正确的容量与限制非常重要。正如前面看到的[docker-compose.yml](#docker-compose)示例，以下选项是必需的:`--cap-add=IPC_LOCK --ulimit memlock=-1:-1 --ulimit nofile=65536:65536`
1. 确保bootstrap.memory_lock被设置为true，原因见[禁用swapping](../Important_System_Configuration/Disable_swapping.md)。你可以参考之前介绍的[配置方式](#configuration-method)，例如通过环境变量采用`-e "bootstrap.memory_lock=true"`设置。
1. 镜像将[expose（发布）](https://docs.docker.com/engine/reference/builder/#/expose) 9200与9300的TCP端口。建议集群通过`--publish-all`发布在随机的端口，除非你在每个机器上只运行一个容器。
1. 使用`ES_JAVA_OPTS`环境变量设置JVM堆大小，例如在`docker run`的时候设置堆大小为16GB：`-e ES_JAVA_OPTS="-Xms16g -Xmx16g"`。还建议设置容器的[内存限制](https://docs.docker.com/engine/reference/run/#user-memory-constraints)。
1. 明确指定你部署的Elasticsearch镜像版本，譬如：`docker.elastic.co/elasticsearch/elasticsearch:5.3.0`。
1. 总是使用绑定的volume（卷）来指定`/usr/share/elasticsearch/data`，就像上面[生产模式](#prpduction-mode)章节所说的那样，主要有一下几个原因：
    * 你不会希望在容器被杀掉时，你的数据丢失。
    * Elasticsearch对I/O要求非常敏感，然而Docker存储驱动并支持高效的I/O。
    * 它允许使用高级的[Docker卷插件](https://docs.docker.com/engine/extend/plugins/#volume-plugins)。
1. 如果你使用的是`devicemapper`存储驱动（最新基于RedHat(rpm)的发型版本默认是这个），请确保不是使用默认的`loop-lvm`模式，配置docker引擎才使用[direct-lvm](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-docker-with-devicemapper)替代它。
1. 考虑采用不同的[日志驱动](https://docs.docker.com/engine/admin/logging/overview/)来集中式管理你的日志。再者说默认的`json-file`日志驱动也不太适合用于生产。

## 下一步

现在，您搭建了一个测试环境Elasticsearch。开始更深入的研究或投入生产使用Elasticsearch之前，你需要做一些额外的配置：

* 了解如何[配置Elasticsearch](../Configuring_Elasticsearch.md)。
* 配置[重要的Elasticsearch设置](../Important_Elasticsearch_configuration.md)。
* 配置[重要的系统设置](../Important_System_Configuration.md)。