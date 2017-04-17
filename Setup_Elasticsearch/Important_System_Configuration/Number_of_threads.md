# 线程数

Elasticsearch不同类的操作使用不同的线程池。在必要的时候创建新的线程非常重要，确保elasticsearch用户可以创建的线程数至少为2048。

这些可以通过root用户在启动前使用[ulimit -u 2048](./Configuring_system_settings.md#ulimit)来设置，或者是在[/etc/security/limits.conf](./Configuring_system_settings.md#limits.conf)中设置`nproc`。