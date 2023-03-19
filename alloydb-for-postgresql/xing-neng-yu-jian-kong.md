# 性能与监控

### AlloyDB 最多支持多少 IOPs？

IOPs 并不是衡量 AlloyDB 性能的最佳方式。AlloyDB 通过减少实例和存储之间的 I/O 来提高性能，因此不能简单的通过比较 IOPs 来判定性能。同样的 workload，在 AlloyDB 上会产生低得多的 I/O。一些对 I/O 负载非常敏感的业务在 AlloyDB 上通常都表现很好。

### AlloyDB 是否像 Cloud SQL一样通过配置存储的大小来配置 IOPs？

AlloyDB 的 IOPs 与存储无关，再加上 AlloyDB 对存储进行自动扩展，因此在配置实例时，也没有配置存储以及 IO 的选项。

### AlloyDB 如何对 page/block 进行持久化？这种架构对性能有什么影响？

提交写入事务时，AlloyDB 会更新其分层缓存（缓冲区缓存和超高速缓存）中的页面，并通过将预写日志 (WAL) 存储在低延迟的区域日志存储中来使事务持久化服务作为提交操作的一部分。之后，这些 WAL 由多线程日志处理服务 (LPS) 异步处理，在底层的分布式存储中持续的将 pages 持久化。更多内容请见[官方博客](https://cloud.google.com/blog/products/databases/alloydb-for-postgresql-intelligent-scalable-storage)。&#x20;

与标准 PostgreSQL 相比，这种架构减少了存储的 I/O 量，因为只有 WAL 记录被发送到存储层。同时，由于多层缓存，AlloyDB 提供了更快的读取速度。读取最近写入的内容可以直接从缓存中提取，在未命中缓存的情况下，才从底层存储拉取。

### AlloyDB 在 read pool 节点上是如何分配 query 的？

使用 round-robin 算法对连向 read pool 的 connections 进行分配。如果是不同的业务需求，可以根据不同的要求创建不同的 read pool。每个 read pool 的节点数，以及 flag，extension 都是独立配置的（所有 read pool 中的总节点数不超过 20）。

### 如何配置合适的 max\_connections flag ？

对于 max\_connections flag，总的限制范围为 1000 到 240000。然而我们知道，每个连接都是需要消耗资源的，不同的机型，配置的内存不同，能够支撑的最大连接数也有所不同。在启动 AlloyDB 时，会预留 100kb 的内存给每一个 connection，因此如果实例所配置的内存较低，max\_connections 设置的又过大，会导致没有足够的内存分配而导致实例无法创建成功。而在实际情况下，一个 connection 所需要的 memory 要比 100kb 多，并发的连接过多会进一步导致数据库 OOM。因此需要根据机型以及实际的工作负载设置合适的 max\_connections的值，而不是越大越好。理论上，对于一个较小的机型（2vCPU， 16G memroy），max\_connections 的值最好不高于 30000。
