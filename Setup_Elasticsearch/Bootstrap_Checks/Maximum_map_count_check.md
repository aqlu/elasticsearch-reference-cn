# 最大map数检查

接着[上一点](./Maximum_size_virtual_memory_check.md)，为了让`mmap`生效，Elasticsearch还需要有创建需要内存映射区的能力。最大map数检查是确保内核允许创建至少262144个内存映射区，只在Linux执行检查。要想通过此检查，你需要通过`sysctl`设置`vm.max_map_count`最少为`262144`。