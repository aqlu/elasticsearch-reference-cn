# 文件描述符检查

文件描述符是Unix架构用来跟踪打开的“files”的。在Unix中，[一些都是文件](https://en.wikipedia.org/wiki/Everything_is_a_file)。例如，“files”可以是物理文件，也可以是虚拟文件（如:`/proc/loadavg`），甚至是一个网络套接字。Elasticserarch需要非常多的文件描述符（譬如：每一个由多个段和文件组成的分片、加上其他节点的连接等等）。此启动检查将在OSX和Linux系统上执行。要通过此项检查，你需要配置[文件描述符](../Important_System_Configuration/File_Descriptors.md)。