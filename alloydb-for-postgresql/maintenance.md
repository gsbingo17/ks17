# Maintenance

### AlloyDB 是否支持 0 宕机维护？

在主实例（Primary），维护动作执行的非常迅速，但是依然会导致短暂的连接中断。AlloyDB 对主实例配置了独立的 standby 以保证集群的高可用。因此维护会先在 standby 实例上执行，然后再与主实例进行 failover。这会导致一个小于 10秒的中断，因此建议在应用端做好 retry 的机制，能够进行重连。

在 read pool 这边，AlloyDB 提供 0 宕机维护（除非 read pool 中只有一个节点）。维护会在 read pool 中的几个节点中轮流进行，因此只要 read pool 中有多个节点，即可保证 read pool 即便在维护时始终可用。节点的维护顺序会根据各个节点的流量进行动态判断，尽量等待节点上的 transaction 运行结束后再进行维护，以减少对于正在执行的语句的中断（除非有特别长的 long-running transaction）。

### 如何管理 AlloyDB 的版本升级?

对于小版本升级 (minor upgrade)，将在维护窗口期间自动升级；大版本升级（major upgrade）由用户自己决定是否升级。
