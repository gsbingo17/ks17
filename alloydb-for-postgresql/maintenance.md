# Maintenance

### AlloyDB 是否支持 0 宕机维护？

在主实例（Primary），维护动作执行的非常迅速，但是依然会导致短暂的连接中断。AlloyDB 对主实例配置了独立的 standby 以保证集群的高可用。因此维护会先在 standby 实例上执行，然后再与主实例进行 failover。这会导致一个小于 10秒的中断，因此建议在应用端做好 retry 的机制，能够进行重连。

在 read pool 这边，AlloyDB 提供 0 宕机维护（除非 read pool 中只有一个节点）。维护会在 read pool 中的几个节点中轮流进行，因此只要 read pool 中有多个节点，即可保证 read pool 即便在维护时始终可用。节点的维护顺序会根据各个节点的流量进行动态判断，尽量等待节点上的 transaction 运行结束后再进行维护，以减少对于正在执行的语句的中断（除非有特别长的 long-running transaction）。

### 如何管理 AlloyDB 的版本升级?

对于小版本升级 (minor upgrade)，将在维护窗口期间自动升级；大版本升级（major upgrade）由用户自己决定是否升级。

### AlloyDB 的备份可以保留多久？如何自定义修改备份规则？

如果是通过 Console 创建的 AlloyDB，在创建时可以通过勾选 Automate backups 来配置每天一次的自动备份，默认的备份保留时间为 14 天。同时，也可以通过 [gcloud](https://cloud.google.com/sdk/gcloud/reference/alloydb/clusters/update?authuser=1) 命令来对备份配置进行自定义修改。例如：

```text
gcloud alloydb clusters update CLUSTER_ID \
    --automated-backup-days-of-week=DAYS_LIST \
    --automated-backup-start-times=05:00 \
    --automated-backup-retention-period=1m \
    --automated-backup-window=90m  \
    --region=REGION_ID \
    --project=PROJECT_ID
```

上边的例子中，DAYS_LIST，可以是一周中的某一个或者几个具体的日子，比如[MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY]

<figure><img src="../.gitbook/assets/Screenshot 2023-03-27 at 17.19.48.png" alt=""><figcaption></figcaption></figure>

修改之后，可以再通过 gcloud describe 命令确认修改后的结果

```text
gcloud alloydb clusters describe test0317 --region <REGION>
```
