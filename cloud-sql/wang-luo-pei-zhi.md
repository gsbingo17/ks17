# 创建实例

### 创建实例是否可以指定相关的数据库参数？

创建实例的时候，是通过[Database Flag](https://cloud.google.com/sql/docs/mysql/flags)来指定参数。最简单的办法是，你可以在Console直接配置Database Flag。

### 能聊聊实例的HA机制吗？

举个例子，Cloud SQL的实例的HA是跨越2个可用区的，其中数据的HA是通过[Regional persistent disks](https://cloud.google.com/compute/docs/disks#repds)来实现，写入到主节点的数据会通过Regional PD实现同时写入到不同的可用区。所以，如果主节点遇到故障，故障切换到在不同的可用区的备用节点上，这个时候数据在这个可用区也是一致和可用的。

HA版本的实例提供了更好的SLA，99.95%。详细信息可以参看这个[文档](https://cloud.google.com/sql/sla)。

### 创建实例的IOPS是如何配置？

IOPS的大小是和磁盘的类型和大小相关的；同时，越大的实例类型，IOPS的上限越大。所以，当你需要高的IOPS的时候，请选择尽量大的实例类型以及SSD磁盘和更大的存储容量。最简单的验证办法是，你可以在Console创建实例时直接配置，并直观的看到IOPS的配置情况。

![](<../.gitbook/assets/image (15).png>)

### 创建实例可以指定使用Public IP和Private IP吗？

Private IP是自动Peering到你的Project的VPC的。配置Public IP的时候，请注意配置Allow List，启用实例的网络访问控制。

### 我可以指定创建的实例的数据库的小版本吗？

使用gcloud的指令创建数据库实例的时候，可以通过[database-version](https://cloud.google.com/sql/docs/mysql/admin-api/rest/v1beta4/SqlDatabaseVersion)指定部分数据库实例的小版本。

### 实例升级配置麻烦吗？

磁盘容量是可以配置自动增长的；如果需要手工修改磁盘容量，实例不需要重新启动。但是增加CPU和内存时候，实例需要重新启动，在重新启动的过程中，实例的停机时间大概在1分钟-2分钟，时间的长短和数据库的繁忙程度相关。

### 我想创建一个Cloud SQL的read replica，能够用更小配置的实例类型节省成本，这个可以吗？

可以的。

### 如果打开Cloud SQL MySQL的Binlog呢？Read replica也可以打开Binlog吗？

打开Cloud SQL MySQL的Binlog，简单的办法是，在Console中选择Backups->Automated Backups->Enable binary logging. 最长可以保留7日的Binlog。\
\
Read replica也可以打开Binlog，简单的办法是:

```
gcloud sql instances patch INSTANCE_NAME \
--enable-bin-log
```

### 有什么好办法可以加速Cloud SQL MySQL的PITR的速度吗

经常使用到办法是，可以使用这个[solution](https://cloud.google.com/sql/docs/mysql/backup-recovery/scheduling-backups)，实现每一个小时做一次On-demand备份；每个小时的备份是增量备份，不会带来明显的额外的备份的成本，但PITR的速度加快，大部分情况下，能在10分钟左右的时间内完成PITR操作。

### 如何确认 Cloud SQL PostgreSQL 的 WAL 是否已经保存在 Google Cloud Storage (GCS) 上了？

1. 查看 `Bytes_used_by_data_type` 指标

如果 archived\_wal\_log 的值为 0，那么意味着实例的 WAL 已经保存在了 GCS 上

2. 登陆到实例， 执行 **EXECUTE SHOW archive\_command;** 语句

如果在结果中看到 "-async\_archive -remove\_storage"，则意味着 WAL 已经保存在 GCS 上。如果看不到该字段，则说明此台实例的 WAL 仍旧保留在实例自身的存储中。

### 保存在 Google Cloud Storage (GCS) 的 WAL log 收否收费？

在 GCS 上的 WAL log（最多可以保留 7 天）的费用，由 GCP 承担，不对用户产生费用。

### 修改Cloud SQL数据库实例的子网，怎么做呢？

现在创建新的实例时，可以**指定实例所在的子网**了；但如果想修改子网呢。举个例子说明具体做法，例如在VPC的Private Service Connections有2个子网，如下图：

![](https://lh6.googleusercontent.com/PZiolpmK\_yl7eVilMvRDlQFacIKdedOGxw7cCyWGLPrrhvrHhzlaTOQKyPaqNjmvr3LNS4zPevPPauyobaM-Mp4DA7WLXiNmaM0RNBFbvwEfQUjKTg0bYspvBjYoM8rSROi\_tB2Z8BEWzsKRl6M3V5iualrh-1ySMQVuAp2lVwJZQevJq6RC6AidiHmul3g)

你想将你的Cloud SQL实例从10.64.0.0/16移动到10.117.0.0/16，应该怎么做呢？先创建一个临时的VPC，并且包括子网，并将这个Cloud SQL实例移动到这个临时的VPC，命令如下：

```
gcloud beta sql instances patch test1 --project=abc \

--network=projects/abc/global/networks/temp \

--no-assign-ip
```

&#x20;然后再将这个实例移动回原来的VPC的子网cloudsql1上，命令如下：

```
  gcloud alpha sql instances patch test1 \
--project=abc \
--network=projects/abc/global/networks/vpn-vpc \
--no-assign-ip --allocated-ip-range-name=cloudsql1
```

### 如果开源数据库某个版本结束生命周期，Cloud SQL还提供支持吗？

举个例子来说明，例如MySQL 5.7将于2023年10月21日结束生命周期，但这不等于Cloud SQL同时结束对MySQL 5.7的支持。当 Cloud SQL 打算结束对特定主要版本的支持时，将至少提前 12 个月发送通知以提醒客户。Cloud SQL也会提供工具，以最大程度减少升级中断；为期 12 个月的期限结束时，任何未迁移到新的主要版本的实例都将自动升级。
