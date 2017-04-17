# 禁用swapping

大多数操作系统尝试使用尽可能多的内存文件系统缓存和热切换出未使用的应用程序内存。这可能导致部分JVM堆被交换到磁盘上。

对于性能和节点的稳定性交换是非常糟糕的，应该不惜一切代价避免。它可能导致垃圾收集持续几**分钟**而不是毫秒，这可能导致节点响应缓慢，甚至脱离集群。

<span id="#mlockall">有三种方式禁用swapping：<span>


## 开启bootstrap.memory_lock

第一步，在Linux与Unix系统中使用[mlockall](http://opengroup.org/onlinepubs/007908799/xsh/mlockall.html)或者在Windows系统中使用[VirtuallLock](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366895%28v=vs.85%29.aspx)来尝试在RAM中锁定进程的地址空间，来阻止Elasticsearch内存被交换出去。

如果这些设置完成，请在`config/elasticsearch.yml`中添加如下行：

```yaml
bootstrap.memory_lock: true
```

> 警告
>
> 如果试图分配的内存大于可用内存，`mlockall`可能导致JVM或shell会话退出！

Elasticsearch启动之后，你可以通过下面请求输出结果中的`mlockall`对应的值来验证是否设置成功：

```js
GET _nodes?filter_path=**.mlockall
```

如果你看到`mlockall`为`false`，那么意味着`mlockall`请求失败了。你也可以在日志文件中通过`Unable to lock JVM Memory`关键词获取更多的信息。

最可能的原因,在Linux与Unix系统上，运行Elasticsearch的用户没有权限锁定内存。这可以通过如下获得:

### `.zip`与`.tar.gz`安装包

&emsp;&emsp;启动Elasticsearch之前通过root设置[ulimit -l ulimited](./Configuring_system_settings.md#ulimit)，或者在[/etc/security/limits.conf](./Configuring_system_settings.md#limits.conf)中设置 **memlock** 为 **unlimited** 。

### RPM与Debian安装包

&emsp;&emsp;在[系统配置](./Configuring_system_settings.md#sysconfig)中设置 **MAX_LOCKED_MEMORY** 为 **ulimited** （或者参见如下 **systemd** 的设置方式）。

### 系统使用`systemd`

&emsp;&emsp;在[systemd配置](./Configuring_system_settings.md#systemd)中设置 **LimitMEMLOCK** 为 **infinity**。

`mlockall`失败的另一个可能的原因是临时目录(通常是`/tmp`)挂载使用`noexec`选项。这可以通过设置`ES_JAVA_OPTS`的环境变量来指定一个新的临时目录：

```bash
export ES_JAVA_OPTS="$ES_JAVA_OPTS -Djava.io.tmpdir=/path/to/temp/dir"
./bin/elasticsearch
```

或者在`jvm.options`配置文件中设置。


## 禁用所有的swap文件

第二种方式就是彻底禁用swap。通常Elasticsearch是运行在一个独立的机器上，且内存使用是通过JVM来控制的。这可能不需要开启swap。

在Linux操作系统，你可以通过`sudo swapoff -a`来临时禁用swap。要永久的停用swap，可以通过编辑`/etc/fstab`文件，注释文件中所有包含swap单词的行。

在Windows操作系统，你可以通过`系统设置->高级->性能->高级->虚拟内存`来禁用分页文件。

## 配置`swappiness`

另一种方式在Linux系统上可用，确保设置`vm.swappiness`为1。这减少了内核交换的倾向和正常情况下不应该导致，同时仍然允许在紧急条件下整个系统交换。