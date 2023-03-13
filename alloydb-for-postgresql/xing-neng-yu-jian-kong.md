# 性能与监控

### AlloyDB 最多支持多少 IOPs？

IOPs 并不是衡量 AlloyDB 性能的最佳方式。AlloyDB 通过减少实例和存储之间的 I/O 来提高性能，因此不能简单的通过比较 IOPs 来判定性能。同样的 workload，在 AlloyDB 上会产生低得多的 I/O。一些对 I/O 负载非常敏感的业务在 AlloyDB 上通常都表现很好。

### AlloyDB 是否像 Cloud SQL一样通过配置存储的大小来配置 IOPs？

AlloyDB 的 IOPs 与存储无关，再加上 AlloyDB 对存储进行自动扩展，因此在配置实例时，也没有配置存储以及 IO 的选项。

### AlloyDB 如何对 page/block 进行持久化？这种架构对性能有什么影响？

提交写入事务时，AlloyDB 会更新其分层缓存（缓冲区缓存和超高速缓存）中的页面，并通过将预写日志 (WAL) 存储在低延迟的区域日志存储中来使事务持久化服务作为提交操作的一部分。之后，这些 WAL 由多线程日志处理服务 (LPS) 异步处理，在底层的分布式存储中持续的将 pages 持久化。更多内容请见[官方博客](https://cloud.google.com/blog/products/databases/alloydb-for-postgresql-intelligent-scalable-storage)。&#x20;

与标准 PostgreSQL 相比，这种架构减少了存储的 I/O 量，因为只有 WAL 记录被发送到存储层。同时，由于多层缓存，AlloyDB 提供了更快的读取速度。读取最近写入的内容可以直接从缓存中提取，在未命中缓存的情况下，才从底层存储拉取。





从最近写入的块中读取通常由缓存提供。当然，在缓存未命中的情况下，读取直接从区域块存储服务。由于这些情况发生在记录更新后足够长的时间，LPS 中的异步后台处理确保 WAL
