# 重要的系统配置

理想情况下，Elasticsearch应该单独在一个服务器上运行，并使用所有可用的资源。为了做到这一点，您需要配置您的操作系统允许用户运行Elasticsearch比默认允许访问更多的资源。

以下设置必须在生产使用时配置:

* [设置JVM堆大小](./Important_System_Configuration/Set_JVM_heap_size_via_jvm.options.md)
* [禁用交换](./Important_System_Configuration/Disable_swapping.md)
* [增加文件描述符](./Important_System_Configuration/File_Descriptors.md)
* [确保足够的虚拟内存](./Important_System_Configuration/Virtual_memory.md)
* [确保足够的线程](./Important_System_Configuration/Number_of_threads.md)



## <span id="dev-vs-prod">开发模式vs生产模式</span>

默认情况下，Elasticsearch假设你在开发环境中工作。任何没有设置正确的配置，都会以警告的方式记录在日志文件中，但是你能启动与运行Elasticsearch节点。

一旦你后面配置了网络设置如`network.host`，Elasticsearch将会切换到生产模式并将警告升级为异常。这些异常将会阻止Elasticsearch节点的启动。这是一个非常重要的安全措施，以确保你不会因为错误的配置丢失数据。