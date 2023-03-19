# 创建实例

### Bigtable Node的性能指标是什么？

和Cloud Spanner类似，Bigtable的Node其实是一个逻辑的Node，不是物理上的机器，这个Node背后是有多个计算节点来提供计算资源以及高可用性的。每个Bigtable的Node都可以提供以下大致的性能，具体取决于使用的存储类型。

![](<../.gitbook/assets/image (29).png>)

一般来说，集群的性能会随着集群中节点数量的增加而线性提升。例如，如果您创建一个具有 10 个节点的 SSD 集群，则在典型的只读或只写工作负载下，该集群最多可以支持每秒 10 万行读取或写入。

### Bigtable提供autoscaling吗？

Bigtable是全托管的数据库云服务。Bigtable提供了autoscaling功能。Bigtable提供了可以按照集群CPU利用率和存储空间利用率目标，在节点数下限和上限之间要么通过扩容向集群添加节点，要么通过缩容从集群中移除节点。autoscaling的好处是既能够保证高峰期的性能，又能降低成本。

### Bigtable中有实例、集群和节点，具体是什么关系呀？

使用Bigtable，首先是创建实例，这个实例可以包括最多8个集群，这些集群可以分不在不同的Zone或者Region；每个集群都有1个或者多个节点；最后，表是创建在实例里面的。

这里稍微特殊的就是集群，集群实际上表示了实例的数据可以分布到不同Zone或Region，数据自动同步，多同样的表可以实现“多读多写”，实现最终一致性，这是一个特性叫Bigtable 全球复制。更多信息，请参考这个[文档](https://cloud.google.com/bigtable/docs/replication-overview?hl=zh-cn)。
