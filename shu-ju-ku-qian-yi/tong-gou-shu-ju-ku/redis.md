---
description: Redis
---

# Redis

### 将Redis迁移到MemoryStore for Redis是容易完成的。分为两种方式：

* 离线批量方式

将源数据库Redis导出，然后通过MemoryStore for Redis的导入功能导入完成数据迁移。导出和导入的数据格式通常为RDB格式。

* 在线实时方式

通常通过[RIOT](https://github.com/redis-developer/riot)或者[Redis-shake](https://github.com/alibaba/RedisShake)等开源工具实现。这些开源工具可以支持使用源数据库Redis的psync或者scan+keyspace notificate来实现数据迁移。优先使用psync；如果源数据库Redis不支持psync，再选择使用scan+keyspace notification。补充一句，MemoryStore for Redis提供了psync的支持。

具体的做法，可以参考上述开源工具的说明文档。
