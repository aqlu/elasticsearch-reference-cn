# 系统调用过滤检查

Elasticsearch安装各自各样的系统调用过滤依赖于操作系统（如：Linux上的seccomp）。如果成功安装与启用了系统调用过滤，将能够防御一些对Elasticsearch调用的攻击行为。要想通过此检查，你必须根据日志来修复任何阻止系统调用过滤的系统错误，或者通过配置`bootstrap.system_call_filter`为`false`来禁用系统调用过滤然后**你自己做风控**。