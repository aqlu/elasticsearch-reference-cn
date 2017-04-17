# 虚拟内存

Elasticsearch默认采用[hybrid mmapfs / niofs](../../Index_Modules/Store.md#default_fs)目录来保存索引。默认的操作系统mmap数限制看起来太小，这可能会导致内存溢出的异常。

在Linux系统，你可以使用`root`用户通过如下命令来增加限制数：

```bash
sysctl -w vm.max_map_count=262144
```

若要永久的生效，可以更新`/etc/sysctl.conf`中的`vm.max_map_count`。重启系统后可以通过运行`sysctl vm.max_map_count`来进行验证。

RPM与Debian安装包会自动设置此配置，不需要进一步配置。