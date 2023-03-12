# 数据库持久性的配置

所有数据都仅在 RAM 或 RAM + 闪存（[闪存上的 Redis](https://docs.redis.com/latest/rs/databases/redis-on-flash/)）中存储和管理，因此在进程或服务器故障时有丢失的风险。由于 Redis Enterprise Software 不仅是一个缓存解决方案，而且还是一个成熟的数据库，因此对磁盘的[持久性](https://redis.com/redis-enterprise/technology/durable-redis/)至关重要。因此，Redis Enterprise Software 支持以每个数据库为基础并以多种方式将数据持久化到磁盘。

可以在数据库创建期间或通过编辑现有数据库的配置来配置[持久性。](https://redis.com/redis-enterprise/technology/durable-redis/)虽然可以动态更改持久性模型，但要知道，数据库从一种持久性模型切换到另一种可能需要一些时间。这取决于您要从什么切换到什么，还取决于您的数据库的大小。

### 为您的数据库配置持久性  <a href="#configure-persistence-for-your-database" id="configure-persistence-for-your-database"></a>

1. 在**数据库**中，要么：
   * 单击**添加**(+) 创建新数据库。
   * 单击要配置的数据库，然后在页面底部单击编辑。
2. 导航到持久性
3. 选择您的数据库持久性选项
4. 选择保存或更新

### 数据持久化选项  <a href="#data-persistence-options" id="data-persistence-options"></a>

Redis Enterprise 软件中有六个持久化选项：

| **选项**           | **描述**            |
| ---------------- | ----------------- |
| 没有任何             | 数据根本不会保存到磁盘。      |
| 每次写入时仅附加文件 (AoF) | 每次写入都会将数据同步到磁盘。   |
| 仅附加文件 (AoF) 一秒钟  | 数据每秒同步到磁盘。        |
| 每 1 小时快照一次       | 每小时创建一次数据库快照。     |
| 每 6 小时快照一次       | 每 6 小时创建一次数据库快照。  |
| 每 12 小时快照一次      | 每 12 小时创建一次数据库快照。 |

### 选择持久性策略  <a href="#selecting-a-persistence-strategy" id="selecting-a-persistence-strategy"></a>

在选择持久性策略时，您应该考虑对数据丢失的容忍度和性能需求。两者之间总会有取舍。fsync() 系统调用将数据从文件缓冲区同步到磁盘。您可以配置 Redis 执行 fsync() 的频率，以最有效地在您的用例的性能和持久性之间进行权衡。Redis 支持三种 fsync 策略：every write、every second 和 disabled。

Redis 还允许通过 RDB 文件进行快照以实现持久化。在 Redis Enterprise 中，您可以配置快照和 fsync 策略。

对于任何高可用性需求，复制也可用于进一步降低数据丢失的风险，强烈推荐使用。

**对于数据丢失成本高的用例：**

1. 仅附加文件 (AOF) - Fsync everywrite - Redis Enterprise 设置开源 Redis 指令`appendfsyncalways`。使用此策略，Redis 将等待写入和 fsync 完成，然后再向客户端发送数据已写入的确认。除了命令的执行之外，这还引入了 fsync 的性能开销。fsync 策略总是有利于持久性而不是性能，并且应该在数据丢失成本很高时使用。

**对于只能有限容忍数据丢失的用例：**

1. Append-only file (AOF) - Fsync 每 1 秒 - Redis 将每秒 fsync 任何新写入的数据。该策略平衡了性能和持久性，并且应该在发生故障时可以接受最小数据丢失的情况下使用。这是默认的 Redis 策略。此策略可能会导致 1 到 2 秒的数据丢失，但平均而言这将更接近一秒。

笔记： 出于性能原因，如果您要使用 AOF，强烈建议确保也为该数据库启用复制。启用这两个功能时，持久化是在数据库副本上执行的，不会影响主服务器的性能。

**对于数据丢失在较长时间内是可以容忍或可以恢复的用例：**

1. 快照，每 1 小时 - 设置每 1 小时一次完整备份。
2. 快照，每 6 小时 - 设置每 6 小时一次完整备份。
3. 快照，每 12 小时 - 设置每 12 小时一次完整备份。
4. 无 - 根本不备份或保留数据。

### 仅追加文件 (AOF) 与快照 (RDB)  <a href="#append-only-file-aof-vs-snapshot-rdb" id="append-only-file-aof-vs-snapshot-rdb"></a>

既然您知道了可用的选项，为了帮助您决定哪个选项适合您的用例，下面是关于这两个选项的表格：

| **仅追加文件 (AOF)**        | **快照（关系数据库）**              |
| ---------------------- | -------------------------- |
| 资源密集度更高                | 资源密集度较低                    |
| 提供更好的持久性（恢复最新的时间点）     | 不太耐用                       |
| 恢复时间较慢（文件较大）           | 更快的恢复时间                    |
| 需要更多磁盘空间（文件往往会变大并需要压缩） | 需要更少的资源（I/O 每几个小时一次，不需要压缩） |

### 双活数据持久化  <a href="#active-active-data-persistence" id="active-active-data-persistence"></a>

Active-Active 数据库仅支持 AOF 持久化。主动-主动数据库不支持快照持久性。

如果 Active-Active 数据库正在使用快照持久化，请使用`crdb-cli`切换到 AOF 持久化：

```
crdb-cli crdb update --crdb-guid <CRDB_GUID> --default-db-config \
   '{"data_persistence": "aof", "aof_policy":"appendfsync-every-sec"}'
```

### Redis on Flash 数据持久化  <a href="#redis-on-flash-data-persistence" id="redis-on-flash-data-persistence"></a>

如果您为在 Redis Enterprise Flash 上运行的数据库启用数据持久性，默认情况下主分片和副本分片都配置为写入磁盘。这不同于标准的 Redis 企业软件数据库，其中只有副本分片持久保存到磁盘。这种带有复制的主从双重数据持久化是为了更好地保护数据库免受节点故障的影响。基于闪存的数据库有望保存更大的数据集，并且在节点故障下分片的修复时间可能会更长。在这些较长的修复时间下，具有双重持久性可以更好地防止故障。

然而，具有复制的双重数据持久性增加了一些处理器和网络开销，特别是在具有网络连接的持久性存储的云配置的情况下（例如 AWS 中的 EBS 支持的卷）。

有时性能对您的用例至关重要，您不想冒数据持久性增加延迟的风险。如果是这种情况，您可以使用以下`rladmin`命令在主分片上禁用数据持久性：

```sh
rladmin tune db <database_ID_or_name> master_persistence disabled

```
