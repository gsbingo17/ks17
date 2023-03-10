# 性能与监控

### 怎么理解Spanner的性能？

Spanner解决了数据库的横向扩展能力，可以随着工作负载的增加，提供性能扩展，并且保证稳定读写延时。举一个通俗的例子，在每秒几百笔的请求下，Cloud Spanner和其他的数据库的性能水平是一样的，但是当每秒请求到了几万笔或者几十万笔的时候，Cloud Spanner还能够提供稳定的性能，可能其他的数据库已经由于无法扩展，而支撑不了这个性能了。

### 如何利用Spanner的性能？

Spanner自动实现数据库层面的分片功能，通过数据库分片达到性能的横向扩展。在具体使用中，需要对表配置主键（Primary Key），表是按照主键进行自动分片。这里的关键的考虑的地方在于：

* 选择合适的Primary Key，可以参考这篇[文档](https://cloud.google.com/spanner/docs/schema-and-data-model#choosing\_a\_primary\_key)。另外，在Schema Design上，这里有一篇博客《[关于 Cloud Spanner，DBA（数据库管理员）需要了解哪些内容？第 1 部分：键和索引](https://www.infoq.cn/article/eiyek5ga1evqr20ttuha)》，值得一读。
* 需要足够多的数据或者负载。分片会基于存储容量和工作负载产生新分片和分片在多个Spanner Node上的均衡。举个通俗的例子，当表之有1个GB的时候，就很难产生新分片；当负载仅仅每秒几百个请求的时候，也很难触发产生新分片，所以当数据量小，负载低的时候，是不能有限产生更多的分片的。举个例子，由于分片少，尽管你有10个node，你也无法用到这个10个node的计算资源的。所以，通常实践是，在大数据量的情况下，例如几百GB，通过提前预热Pre-warming，达到产生足够的分片，并均衡到所有的node上的效果。这在一些基于Spanner开发的大型应用上线的时候广泛使用，可以参考这篇[文档](https://cloud.google.com/blog/products/databases/cloud-spanner-makes-application-launches-easier-with-warmup-and-benchmarking-tool)。

### Spanner的Interleaved Table是干嘛的呀？



### Spanner提供了哪些具体的监控信息吗？

* System insights
* Query insights
* Transaction insights
* Lock insights

### Spanner提供执行计划吗？

当然有呀！详细信息请参考这个文档。
