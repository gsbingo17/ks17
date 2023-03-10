---
description: Maintenance
---

# Maintenance

### 我能让Cloud SQL Instances不做Maintenance吗？

可以的。你可以在Console或者使用命令行配置Deny maintenance period，每次最多可以配置到90天；快到期的时候，还可以继续配置下一个最多90天的Deny maintenance period。所以，理论上你是可以通过Deny maintenance period长期关闭maintenance的，但是我们建议一年内至少做一次maintenance，通过这个maintenance使得你的instance获得最新的特性和各类补丁。\
如果你现在收到了maintenance的通知，如果你现在配置Deny maintenance period，这个当前的maintenance也会关闭。详细的操作，请参考这个[文档](https://cloud.google.com/sql/docs/mysql/set-maintenance-window#configure-deny-maintenance)。

### 我能选择一个任意的时间对我的Cloud SQL Instance做Maintenance吗？

可以的。Cloud SQL提供了自助服务的maintenance。详细的操作，请参考这个[文档](https://cloud.google.com/sql/docs/mysql/self-service-maintenance)。

### 我能为Cloud SQL配置maintenance窗口吗？

可以的。

