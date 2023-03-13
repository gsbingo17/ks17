# 创建实例

### 目前有哪些Redis的部署是可以选择的？

* 支持的版本3、4、5、6.x，默认版本是6.x
* Redis的单节点部署：选择MemoryStore Redis的基础版，通常用于开发测试
* Redis的主从高可用性部署：选择MemoryStore Redis的标准版本，通常用于生产环境；标准版提供基于Redis Replication的主从数据同步、一到五个读取副本、自动故障切换等

### 选择MemoryStore Redis的实例配置的时候，有什么其他注意的事情？

* MemoryStore的网络带宽是给的比较充足的，具体可以参考这个[文档](https://cloud.google.com/memorystore/docs/redis/pricing?hl=zh-cn)
* MemoryStore仅仅提供私有网络访问，所以，很多老习惯是不启用Auth也是支持的
* MemoryStore的数据同步是基于Redis的Replication，这个Replication本身就是异步的，复制的异步特性导致副本可能会滞后于主节点，具体取决于主节点上的写入速率；所以Redis的主从复制是不能完全保证没有数据丢失的。MemoryStore做了一些增强，当有多个副本节点的时候会自动故障切换到具有最小复制延迟的健康副本，尽量缩小可能的数据丢失
* 作为一个全托管服务，MemoryStore提供了部分Redis的可修改的[参数配置](https://cloud.google.com/memorystore/docs/redis/supported-redis-configurations?hl=zh-cn)，请酌情修改。同时，你也会发现有些Redis命令使用不了，这个都是云服务通用的实践，具体哪些命令使用不了，请参看这个[文档](https://cloud.google.com/memorystore/docs/redis/product-constraints?hl=zh-cn)。友情提示，psync是可以用的
* MemoryStore提供了扩容/缩容，版本升级等功能；举个例子，内存从4GB升级到8GB，Redis 5升级到Redis 6；如果使用标准版本，这些操作是轮流方式实现的，尽量减少实例不可用的时间。例如先做从节点，主从切换后，在做主节点，同时对外提供访问的IP地址始终不变的。
