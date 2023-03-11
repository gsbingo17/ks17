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

可以的，具体的步骤可以看看这个[文档](https://cloud.google.com/sql/docs/mysql/set-maintenance-window?hl=zh-cn#opt-in)。并且你会在至少提前一周收到Maintenance的电子邮件的通知，电子邮件的通知方式需要自己配置，请参考下图；当你收到了通知，你可以选择1）立即做maintenance；2）重新调度到下一个maintenance窗口，你可以多次重新安排，只要在原计划的时间之后的28天之内。

![](https://lh6.googleusercontent.com/LGSxteOyEnU-2Kfd5v2pYVw4U5CZ6BatFeHle6Hfd6942vncV-KAgGWK9RVa5y1SiT8oy81wtigPiGlp8kL4oJNRrJjl5gNdKV76dbjjHQzyQb\_-byfFjdHfiS81cK1C7m57eIlAtTRKjjDqwocf\_r4dVhz4PYpq4jONTRSi8YdR-\_5MLxT03JYSmajLPvQTEkz0I\_hXMDskKJMUzxs2Ql9-jj49AF116QFC)

### Maintenance到底做了啥？

Maintenance具体做的事情分几个部分：发布新的功能，修改bug，对数据库版本打各类补丁。Cloud SQL也提供了[Maintenance的日志](https://cloud.google.com/sql/maintenance-changelog?hl=zh-cn)，供大家查看。

### 维护期间 Cloud SQL 实例连接丢失的时间是在什么范围？

* PostgreSQL - 30 seconds or less
* MySQL - 60 seconds or less
* SQL Server - 120 seconds or less

为了优化这个时间，请尽量选择负载低峰期完成，同时也需要优化应用的重试的代码。



