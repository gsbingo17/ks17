# 导出时实例CPU 利用率过高

### Redis 导出原理：

Memorystore for Redis export 使用 Redis 的 BGSAVE 功能对实例中的数据进行快照。执行 BGSAVE 时，Redis 会生成一个新的子进程来进行快照。Redis 在此过程中会使用[写入时复制](https://redis.io/topics/persistence)\[COW]。

对于所有 Redis 实例（包括已启用读取副本的实例），数据都会从主节点导出。因此，导出时Primary node 实例的CPU 使用率会明显增加；

### MemoryStore for Redis Export 期间影响：

* export 期间可以正常读写实例，不能对实例进行配置管理操作，如扩缩实例、更新实例配置等
* 在export 时，实例可能会遇到延迟增加
* export 过程中可以取消
  * 取消导出会删除正在写入 Cloud Storage 存储分区的 RDB 文件，并释放 BGSAVE 进程使用的内存。

### 解决方式

使用标准版的MemoryStore，并启用RDB快照；快照不是在主节点做的，而是在从节点做的，所以对主节点没有影响。快照也实现备份功能。
