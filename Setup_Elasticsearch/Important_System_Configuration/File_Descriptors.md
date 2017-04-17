# 文件描述符

> 注意
>
> 此设置仅需要在Linux或maxOSd的环境中配置，Windows系统可以忽略。在Windows系统中，JVM[API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858\(v=vs.85\).aspx)限制仅受限于可用的资源。

Elasticsearch使用大量的文件描述符或文件句柄。文件描述符超限在运行时灾难性的，很可能导致数据丢失。请确保调大运行Elasticsearchd的用户允许打开文件描述符数量到65536或更大。

如果使用的是`.zip`与`.tar.gz`安装包，在启动elasticsearch前通过`root`用户设置[ulimit -n 65536](./Configuring_system_settings.md#ulimit)，或者是设置[/etc/security/limits.conf](./Configuring_system_settings.md#limits.conf)的`nofile`为`65536`。

RPM和Debian软件包已经默认文件描述符的最大数量为65536，不需要进一步配置。

你可以通过各节点的[Nodes Stats](../../Cluster_APIs/Cluster_Stats.md)API来检查`max_file_descriptions`:

```js
GET _nodes/stats/process?filter_path=**.max_file_descriptors
```