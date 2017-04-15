# Elasticsearch设置

本节包括有关如何设置Elasticsearch并使其运行的信息，包括：

* 下载
* 安装
* 启动
* 配置

## 支持的平台

官方支持的操作系统和JVM的矩阵可以查看这里： [Support Matrix(支持矩阵)](https://www.elastic.co/support/matrix)。 Elasticsearch在列出的平台上测试，但它也可能能够在其他平台上工作。

## Java(JVM) 版本

Elasticsearch是使用Java构建的，并且至少需要[Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)才能运行。 只支持Oracle的Java和OpenJDK。 在所有Elasticsearch节点和客户端上应使用相同的JVM版本。

我们建议安装Java版本**1.8.0_73*或更高版本**。 如果使用已知的错误版本的Java，Elasticsearch将拒绝启动。

Elasticsearch使用的Java版本可以通过设置`JAVA_HOME`环境变量进行配置。

> 注意
>
> Elasticsearch自带的JVM默认配置是运行在64位服务器上的， 如果你想以客户端模式运行在32位机器上，你需要在[jvm.options](./Setup_Elasticsearch/Important_System_Configuration/Configuring_system_settings.md#jvm-options)配置文件中删除`-server`参数，以及无论是在32位客户机还是服务器上运行Elasticsearch，你都需要重新配置线程堆大小在`-Xss1m`与`-Xss320k`之间
