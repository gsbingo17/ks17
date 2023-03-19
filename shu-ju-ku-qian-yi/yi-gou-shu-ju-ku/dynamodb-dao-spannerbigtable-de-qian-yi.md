# DynamoDB到Spanner/Bigtable的迁移

DynamoDB是一个广受欢迎的 Key-Value NoSQL数据库，如果考虑数据库迁移，在GCP数据库里面有几个主要选择。

* 如果仅使用DynamoDB的基本功能，可以考虑将DynamoDB迁移到Bigtable。
* 如果使用DynamoDB的一些高级用法，例如Secondary index、List/Map/Set这些数据类型，可以考虑将DynamoDB迁移到Spanner更为合适，因为Spanner提供了Secondary Index的支持，以及提供了更为丰富的数据库数据类型。
* 补充说明一下，也有将DynamoDB迁移到MongoDB实践做法。
