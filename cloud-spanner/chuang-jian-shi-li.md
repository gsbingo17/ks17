# 创建实例

### Spanner Node的性能指标是什么？

创建Spanner的实例的时候，只需要指定Node的数目来满足性能的要求。因为不知道每个Node的计算资源多少，过去通常是说你需要多少CPU和多少内存，在这里就没有办法找到对应的配置了。

所以，在这里通俗解释一下Node的性能指标。这里的Node其实是一个逻辑的Node，不是物理上的机器，这个Node背后是有多个计算节点来提供计算资源以及高可用性的。1个Node相当于具备1,000个process units，通过使用标准的基准测试工具（YCSB）测试取得性能指标，例如单个region的配置下1个Node可以支撑每秒15,000笔读或者每秒2,300笔写。可以初步按照这个来做一些基本的Sizing。由于Spanner提供很好的扩展性，所以可以非常容易增加Node甚至减少Node来匹配真实的工作负载。详细的信息可以参考这个[文档](https://cloud.google.com/spanner/docs/performance)。

### 单region和多region的配置使用场景是什么？

单region的配置是使用最多的场景，在主要的GCP的region都可以部署单region的配置的Spanner instance。目前客户使用单region的配置的Spanner instance居多。

选择多region的配置的场景，主要是基于容灾的考虑，多region的配置提供更高的SLA。多region的配置目前来看是预先配置好，以供用户选择。多region的配置当中有一个大洲内的配置，例如在美东的多region的配置，也有跨大洲的配置，例如有跨北美、欧洲、亚洲的配置。多region的配置相对于单region的配置由于提升了SLA，性能会有额外开销。详细的信息可以参考这个[文档](https://cloud.google.com/spanner/docs/instance-configurations)。

多region的配置不是用于优化就近读写的性能，这一点值得注意。举个例子，假设Spanner的配置是跨region A和region B的，其中region A是[leader region](https://cloud.google.com/spanner/docs/modifying-leader-region)，这个时候，在region B对Spanner的读（强读）和写操作通常不会带来更好的性能，例外的地方是，stale read有性能的大幅提升。

### Spanner提供定制化的配置吗？

Spanner部分提供了定制化的配置，可以在现有的配置下，增加read replica，提升在就近stale read的性能。例如，你可以创建一个跨香港和新加坡的配置，香港的node可读可写，新加坡的node是一个读的replica，这样子的配置提升了新加坡的stale read的性能。你可以直接在Console上创建，详细的信息可以参考这个[文档](https://cloud.google.com/blog/products/databases/introducing-spanner-configurable-read-only-replicas)。

### Spanner有啥成本低的环境可以用于测试和开发吗？

当然有的！

* 提供90天的Free Trial
* 有最小的100 processing units的instance可以使用，相当于1/10个node的配置，费用一个月大概几十美元。用好的话，还可以直接向上升级配置。
* 有在笔记本电脑就可以使用的[Spanner模拟器](https://cloud.google.com/spanner/docs/emulator?hl=zh-cn)，是一个基于容器镜像。

### GoogleSQL方言和PostgreSQL方言到底选啥？

创建完实例，下一步创建数据库。创建数据库会让你选择是用GoogleSQL还是PostgreSQL方言的数据库。缺省我们都选择GoogleSQL方言的数据库；如果你是从PostgreSQL数据库的开发转过来的，例如，你想直接使用PostgreSQL的DDL/DML，例如直接用PG的DDL来建表建索引，那你就选择PostgreSQL方言的数据库。现在Spanner对PostgreSQL的兼容支持更好一些，除了有PostgreSQL方言支持，还有专门兼容PostgreSQL的Spanner的客户端驱动。

### 现在Spanner和MySQL的兼容的情况如何？

Spanner现在是没有完全兼容MySQL的。举个通俗的例子，已经基于MySQL写的应用，不能直接切换数据库的指向就切换到Spanner上。但由于Spanner也提供标准的SQL和事务，支持大多数的ORM组件，可以说某种程度兼容了MySQL，还是前面的这个例子，假设这个基于MySQL的应用是用MyBatis写的，你可以通过一些开源工具软件，例如[Harbourbridge](https://github.com/cloudspannerecosystem/harbourbridge)，将MySQL里面的库表结构和数据自动转化和迁移到Spanner，这样子基本完成数据库层面的改造和迁移；然后，应用代码切换到Spanner的JDBC的驱动，生成新的MyBatis Mapper，检查正常后，就基本完成了应用的改造了。所以，解决兼容性的问题也可以很容易。

当然，目前有一些MySQL的特性，例如自增ID和存储过程，这些在Spanner里面是没有，要解决这些差异，需要更多的工作量。例如自增ID可以用UUID代替，很多ORM内置了UUID生成器。
