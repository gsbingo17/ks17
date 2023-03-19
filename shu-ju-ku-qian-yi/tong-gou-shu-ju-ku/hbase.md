# HBase

从HBase到Bigtable的数据迁移，可以看作为同构数据库之间的迁移。通常的做法有：

* 基于导出和导入工具实现

例如导出为Avro、Parquet、SequenceFile、CSV，然后使用Dataflow导入到Bigtable。这些都有一些[现有的工具](https://cloud.google.com/bigtable/docs/import-export?hl=zh-cn)可以用。有些时候导出为HFile，这个时候用Spark导入Bigtable更方便一些，因为Spark可以直接读取和解析HFile。

* 基于HBase的快照的导出和导入

使用快照后，整个数据迁移速度会加快。详细步骤可以参考这个[文档](https://cloud.google.com/architecture/hadoop/hadoop-gcp-migration-data-hbase-to-bigtable?hl=zh-cn)。

* 介于HBase的Replication来实现迁移

借助开源的Cloud Bigtable HBase的复制库实现数据库迁移。具体做法可以参考这个[文档](https://cloud.google.com/bigtable/docs/hbase-replication?hl=zh-cn)。
