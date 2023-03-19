# 数据源

### Datastream目前支持哪些数据源？

Datastream是一个无服务器、变动数据捕捉（CDC）和复制的云服务，可以在异构的数据库之间同步数据，同时保持稳定性和较低复制延时。

目前，Datastream支持的数据库源包括

* MySQL
* PostgreSQL
* Oracle，使用[Oracle Logminer](https://docs.oracle.com/en/database/oracle/oracle-database/18/sutil/oracle-logminer-utility.html#GUID-3417B738-374C-4EE3-B15C-3A66E01AE2B5)获取数据变更

详细的信息可以参考这个[文档](https://cloud.google.com/datastream/docs/sources?hl=zh-cn)。

### 如何配置 AWS RDS PostgreSQL 作为 Datastream 的源端

* 设置`logical_replication` = 1 （该参数默认值为 0）
* `max_slot_wal_keep_size` 如果是 PostgrdSQL 13 版本以上，建议同时设置 max\_slot\_wal\_keep\_size 参数，以限制 replication slots 会使用的存储的量。
* `wal_sender_timeout` = 0

如果要创建 10 条以上 stream，或者有其他的下有任务需要用到 logical replication slots 并且总和数量超过 10 的，需要修改以下参数：

* `max_slot_wal_keep_size` 将这个参数调大，每条stream 需要 1 个 replication slot
* `max_wal_senders` 这个参数的值需要大于 `max_slot_wal_keep_size`
